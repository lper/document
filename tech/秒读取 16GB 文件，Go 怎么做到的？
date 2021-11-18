当今世界的任何计算机系统每天都会生成大量的日志或数据。随着系统的发展，将调试数据存储到数据库中是不可行的，因为它们是不可变的，并且只能用于分析和解决故障。所以大部分公司倾向于将日志存储在文件中，而这些文件通常位于本地磁盘中。

  

我们将使用 Go 语言，从一个大小为 16GB 的. txt 或. log 文件中提取日志。

  

让我们开始编码……

  

首先，我们打开文件。对于任何文件的 IO，我们都将使用标准的 Go os.File。

```
f, err := os.Open(fileName)
 if err != nil {
   fmt.Println("cannot able to read the file", err)
   return
 }
// UPDATE: close after checking error
defer file.Close()  //Do not forget to close the file
```

打开文件后，我们有以下两个选项可以选择：

  

*   逐行读取文件，这有助于减少内存紧张，但需要更多的时间。
    
*   一次将整个文件读入内存并处理该文件，这将消耗更多内存，但会显著减少时间。
    

  

由于文件太大，即 16 GB，因此无法将整个文件加载到内存中。但是第一种选择对我们来说也是不可行的，因为我们希望在几秒钟内处理文件。

  

但你猜怎么着，还有第三种选择。瞧…… 相比于将整个文件加载到内存中，在 Go 语言中，我们还可以使用 bufio.NewReader() 将文件分块加载。

```
r := bufio.NewReader(f)
for {
buf := make([]byte,4*1024) //the chunk size
n, err := r.Read(buf) //loading chunk into buffer
   buf = buf[:n]
if n == 0 {
   
     if err != nil {
       fmt.Println(err)
       break
     }
     if err == io.EOF {
       break
     }
     return err
  }
}
```

一旦我们将文件分块，我们就可以分叉一个线程，即 Go routine，同时处理多个文件区块。上述代码将修改为：

```
//sync pools to reuse the memory and decrease the preassure on Garbage Collector
linesPool := sync.Pool{New: func() interface{} {
        lines := make([]byte, 500*1024)
        return lines
}}
stringPool := sync.Pool{New: func() interface{} {
          lines := ""
          return lines
}}
slicePool := sync.Pool{New: func() interface{} {
           lines := make([]string, 100)
           return lines
}}
r := bufio.NewReader(f)
var wg sync.WaitGroup //wait group to keep track off all threads
for {
     
     buf := linesPool.Get().([]byte)
     n, err := r.Read(buf)
     buf = buf[:n]
if n == 0 {
        if err != nil {
            fmt.Println(err)
            break
        }
        if err == io.EOF {
            break
        }
        return err
     }
nextUntillNewline, err := r.ReadBytes('\n')//read entire line
     
     if err != io.EOF {
         buf = append(buf, nextUntillNewline...)
     }
     
     wg.Add(1)
     go func() { 
      
        //process each chunk concurrently
        //start -> log start time, end -> log end time
        
        ProcessChunk(buf, &linesPool, &stringPool, &slicePool,     start, end)
wg.Done()
     
     }()
}
wg.Wait()
}
```

上面的代码，引入了两个优化点：

  

*   sync.Pool 是一个强大的对象池，可以重用对象来减轻垃圾收集器的压力。我们将重用各个分片的内存，以减少内存消耗，大大加快我们的工作。
    
*   Go Routines 帮助我们同时处理缓冲区块，这大大提高了处理速度。
    

  

现在让我们实现 ProcessChunk 函数，它将处理以下格式的日志行。

```
2020-01-31T20:12:38.1234Z, Some Field, Other Field, And so on, Till new line,...\n
```

我们将根据命令行提供的时间戳提取日志。

```
func ProcessChunk(chunk []byte, linesPool *sync.Pool, stringPool *sync.Pool, slicePool *sync.Pool, start time.Time, end time.Time) {
//another wait group to process every chunk further                             
      var wg2 sync.WaitGroup
logs := stringPool.Get().(string)
logs = string(chunk)
linesPool.Put(chunk) //put back the chunk in pool
//split the string by "\n", so that we have slice of logs
      logsSlice := strings.Split(logs, "\n")
stringPool.Put(logs) //put back the string pool
chunkSize := 100 //process the bunch of 100 logs in thread
n := len(logsSlice)
noOfThread := n / chunkSize
if n%chunkSize != 0 { //check for overflow 
         noOfThread++
      }
length := len(logsSlice)
//traverse the chunk
     for i := 0; i < length; i += chunkSize {
         
         wg2.Add(1)
//process each chunk in saperate chunk
         go func(s int, e int) {
            for i:= s; i<e;i++{
               text := logsSlice[i]
if len(text) == 0 {
                  continue
               }
           
            logParts := strings.SplitN(text, ",", 2)
            logCreationTimeString := logParts[0]
            logCreationTime, err := time.Parse("2006-01-  02T15:04:05.0000Z", logCreationTimeString)
if err != nil {
                 fmt.Printf("\n Could not able to parse the time :%s       for log : %v", logCreationTimeString, text)
                 return
            }
// check if log's timestamp is inbetween our desired period
          if logCreationTime.After(start) && logCreationTime.Before(end) {
          
            fmt.Println(text)
           }
        }
        textSlice = nil
        wg2.Done()
     
     }(i*chunkSize, int(math.Min(float64((i+1)*chunkSize), float64(len(logsSlice)))))
   //passing the indexes for processing
}  
   wg2.Wait() //wait for a chunk to finish
   logsSlice = nil
}
```

对上面的代码进行基准测试。以 16 GB 的日志文件为例，提取日志所需的时间约为 25 秒。

  

完整的代码示例如下：

```
func main() {

 s := time.Now()
 args := os.Args[1:]
 if len(args) != 6 { // for format  LogExtractor.exe -f "From Time" -t "To Time" -i "Log file directory location"
  fmt.Println("Please give proper command line arguments")
  return
 }
 startTimeArg := args[1]
 finishTimeArg := args[3]
 fileName := args[5]

 file, err := os.Open(fileName)
 
 if err != nil {
  fmt.Println("cannot able to read the file", err)
  return
 }
 
 defer file.Close() //close after checking err
 
 queryStartTime, err := time.Parse("2006-01-02T15:04:05.0000Z", startTimeArg)
 if err != nil {
  fmt.Println("Could not able to parse the start time", startTimeArg)
  return
 }

 queryFinishTime, err := time.Parse("2006-01-02T15:04:05.0000Z", finishTimeArg)
 if err != nil {
  fmt.Println("Could not able to parse the finish time", finishTimeArg)
  return
 }

 filestat, err := file.Stat()
 if err != nil {
  fmt.Println("Could not able to get the file stat")
  return
 }

 fileSize := filestat.Size()
 offset := fileSize - 1
 lastLineSize := 0

 for {
  b := make([]byte, 1)
  n, err := file.ReadAt(b, offset)
  if err != nil {
   fmt.Println("Error reading file ", err)
   break
  }
  char := string(b[0])
  if char == "\n" {
   break
  }
  offset--
  lastLineSize += n
 }

 lastLine := make([]byte, lastLineSize)
 _, err = file.ReadAt(lastLine, offset+1)

 if err != nil {
  fmt.Println("Could not able to read last line with offset", offset, "and lastline size", lastLineSize)
  return
 }

 logSlice := strings.SplitN(string(lastLine), ",", 2)
 logCreationTimeString := logSlice[0]

 lastLogCreationTime, err := time.Parse("2006-01-02T15:04:05.0000Z", logCreationTimeString)
 if err != nil {
  fmt.Println("can not able to parse time : ", err)
 }

 if lastLogCreationTime.After(queryStartTime) && lastLogCreationTime.Before(queryFinishTime) {
  Process(file, queryStartTime, queryFinishTime)
 }

 fmt.Println("\nTime taken - ", time.Since(s))
}

func Process(f *os.File, start time.Time, end time.Time) error {

 linesPool := sync.Pool{New: func() interface{} {
  lines := make([]byte, 250*1024)
  return lines
 }}

 stringPool := sync.Pool{New: func() interface{} {
  lines := ""
  return lines
 }}

 r := bufio.NewReader(f)

 var wg sync.WaitGroup

 for {
  buf := linesPool.Get().([]byte)

  n, err := r.Read(buf)
  buf = buf[:n]

  if n == 0 {
   if err != nil {
    fmt.Println(err)
    break
   }
   if err == io.EOF {
    break
   }
   return err
  }

  nextUntillNewline, err := r.ReadBytes('\n')

  if err != io.EOF {
   buf = append(buf, nextUntillNewline...)
  }

  wg.Add(1)
  go func() {
   ProcessChunk(buf, &linesPool, &stringPool, start, end)
   wg.Done()
  }()

 }

 wg.Wait()
 return nil
}

func ProcessChunk(chunk []byte, linesPool *sync.Pool, stringPool *sync.Pool, start time.Time, end time.Time) {

 var wg2 sync.WaitGroup

 logs := stringPool.Get().(string)
 logs = string(chunk)

 linesPool.Put(chunk)

 logsSlice := strings.Split(logs, "\n")

 stringPool.Put(logs)

 chunkSize := 300
 n := len(logsSlice)
 noOfThread := n / chunkSize

 if n%chunkSize != 0 {
  noOfThread++
 }

 for i := 0; i < (noOfThread); i++ {

  wg2.Add(1)
  go func(s int, e int) {
   defer wg2.Done() //to avaoid deadlocks
   for i := s; i < e; i++ {
    text := logsSlice[i]
    if len(text) == 0 {
     continue
    }
    logSlice := strings.SplitN(text, ",", 2)
    logCreationTimeString := logSlice[0]

    logCreationTime, err := time.Parse("2006-01-02T15:04:05.0000Z", logCreationTimeString)
    if err != nil {
     fmt.Printf("\n Could not able to parse the time :%s for log : %v", logCreationTimeString, text)
     return
    }

    if logCreationTime.After(start) && logCreationTime.Before(end) {
     //fmt.Println(text)
    }
   }
   

  }(i*chunkSize, int(math.Min(float64((i+1)*chunkSize), float64(len(logsSlice)))))
 }

 wg2.Wait()
 logsSlice = nil
}
```

**原**文链接：https://medium.com/swlh/processing-16gb-file-in-seconds-go-lang-3982c235dfa2
