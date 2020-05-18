> 走进 Golang 之 Channel 的使用

对于 Golang 语言应用层面的知识，先讲如何正确的使用，然后再讲它的实现。

channel 是什么
===========

> Don't communicate by sharing memory, share memory by communicating.

相信写过 Go 的同学都知道这句名言，可以说 channel 就是后边这句话的具体实现。我们来看一下到底 channel 是什么？

channel 是一个类型安全的队列（循环队列），能够控制 groutine 在它上面读写消息的行为，比如：阻塞某个 groutine ，或者唤醒某个 groutine。

不同的 groutine 可以通过 channel 交换任意的资源，由于 channel 能够控制 groutine 的行为，所以 CSP 模型才能在 Golang 中顺利实现，它确保了不同 groutine 之间的数据同步机制。

上面的话是不是听起来非常的不舒服？

好吧，简单说人话就是，channel 是用来在 **不同的** 的 goroutine 中交换数据的。一定要注意这里 **不同的** 三个字。千万不要把 channel 拿来在不同函数（同一个 goroutine 中）间交换数据。

使用
==

知道了定义，我们来看具体如何使用。

如何定义一个 channel 类型呢？

```
var ch1 chan int // 定义了一个 int 类型的 channel，没有初始化，是 nil

ch2 := make(chan int) // 定义+初始化了一个无缓冲的 int 类型 channel
ch3 := make(chan int) // 定义+初始化了一个有缓冲的 int 类型 channel
复制代码

```

上面的定义方法我们都是定义的双向通道，对应的还有单向通道，但是单向通道我们一般只是做为函数参数来进行一些限制，并不会在定义、初始化时就搞一个单向通道出来。因为你定义一个单向通道没有任何实际价值，通道的存在本来就是用来交换数据的，单向通道只能满足发或者收。

下面我们一起来看一下具体的使用，以及使用中注意的一些点。

send
----

不管是有缓冲的通道还是无缓冲的通道都是用来交换数据的，既然是交换数据，无非就是写入、读取。我们先从发送开始。

### 无缓冲 channel

```
ch := make(chan int)
defer close(ch)

//ch<-5 // 位置一

go func(ch chan int) {
    num := <-ch
    fmt.Println(num)
}(ch)

// ch<-5 // 位置二

复制代码

```

如果我们打开 **位置一** 的注释，程序是无法获得预期执行的，由于该 channel 是无缓冲的，位置一的代码会陷入阻塞，下一行的 goroutine 根本没有机会执行。整个代码会陷入死锁。

正确的操作是，打开 **位置二** 的注释，因为上一行 goroutine 先行启动，他是一个独立的协程，不会阻塞主 groutine 的执行，但它内部会阻塞在 `num := <-ch` 这行代码，直到主协程执行完 `ch<-5` ，才会执行打印。所以这里也有一个非常重要的问题，主协程如果不等待子协程执行完就退出的话，会看不到执行结果。

这里先提一点，无缓冲的 channel 并不会用到内部结构体的 `buf` ，这部分具体会在源码部分讲解他们的数据存取、交换的方式。

### 有缓冲 channel

```
ch := make(chan int, 1) // 注意这里
defer close(ch)

//ch<-5 // 位置一

go func(ch chan int) {
    num := <-ch
    fmt.Println(num)
}(ch)

// ch<-5 // 位置二
复制代码

```

代码基本没有改变，唯一的区别是 make 函数传入了第二个参数，这个值的含义是缓冲的大小。那么此时 **位置一** 与 **位置二** 都能够正常执行吗？

答案是肯定的，此时的代码，无论是那个位置，打开注释后都能够正常执行。原因就在于由于 channel 有了缓存区域，**位置一** 写入数据不会造成主协程的阻塞，那么下一行代码的子协程就可以正常启动，并直接将位置一写入 `buf` 的数据读取出来打印。

对于 **位置二** ，由于子协程先启动，但是会被阻塞在 `num := <-ch` 这一行，因为此时 `buf` 中没有任何内容可读取（下期源码分析我们可以看代码实现），直到位置二执行完，唤醒子协程。

发送需要注意几个问题：

1.  什么时候会被阻塞？
    
    *   向 `nil` 通道发送数据会被阻塞
        
    *   向无缓冲 channel 写数据，如果读协程没有准备好，会阻塞
        
    *   向有缓冲 channel 写数据，如果缓冲已满，会阻塞
        
    
2.  什么时候会 `panic`？
    
    *   closed 的 channel，写数据会 panic
        
    
3.  就算是有缓冲的 channel ，也不是每次发送、接收都要经过缓存，如果发送的时候，刚好有等待接收的协程，那么会直接交换数据。
    

receive
-------

有写入，必然后读取。

还是上面的代码， `num := <-ch` 就是从 channel 读取数据。对于读取就不按照有缓冲与无缓冲来讲解了，它们的主要问题是什么时候阻塞。通过上面写的例子自己再想想即可。

这里说下读取的两种形式。

**形式一**

> multi-valued assignment

```
v, ok := <-ch
复制代码

```

`ok` 是一个 bool 类型，可以通过它来判断 channel 是否已经关闭，如果关闭该值为 true ，此时 v 接收到的是 channel 类型的零值。比如： channel 是传递的 int， 那么 v 就是 0 ；如果是结构体，那么 v 就是结构体内部对应字段的零值。

**形式二**

```
v := <-ch
复制代码

```

该方式对于关闭的 channel 无法掌控，我们示例中就是该种方式。

接收需要注意几个问题：

1.  什么时候会被阻塞？
    
    *   从 `nil` 通道接收数据会被阻塞
        
    *   从无缓冲 channel 读数据，如果写协程没有准备好，会阻塞
        
    *   从有缓冲 channel 读数据，如果缓冲为空，会阻塞
        
    
2.  读取的 channel 如果被关闭，并不会影响正在读的数据，它会将所有数据读取完毕，并不会立即就失败或者返回零值
    

close
-----

对于 channel 的关闭，在什么地方去关闭呢？因为上面也讲到向 closed 的 channel 写或者继续 close 都会导致 panic 问题。

一般的建议是谁写入，谁负责关闭。如果涉及到多个写入的协程、多个读取的协程？又该如何关闭？总的来说就是加入一个标记避免重复关闭。不过真的不建议搞的太复杂，否则后续维护代码会疯掉。

关闭需要注意几个问题：

1.  什么时候会 `panic`？
    
    *   closed 的 channel，再次关闭 close 会 panic
        
    

for-range
---------

我们常常会用 `for-range` 来读取 channel 的数据。

```
ch := make(chan int, 1)

go func(ch chan int) {
    for i := 0; i < 10; i++ {
        ch <- i
    }
    close(ch)
}(ch)

for val := range ch {
    fmt.Println(val)
}
复制代码

```

该语句的一个特色是如果 channel 已经被关闭，它还是会继续执行，直到所有值被取完，然后退出执行。而如果通道没有关闭，但是 channel 没有可读取的数据，它则会阻塞在 range 这句位置，直到被唤醒。但是如果 channel 是 nil，那么同样符合我们上面说的的原则，读取会被阻塞，也就是会一直阻塞在 `range` 位置。

select
------

`select` 是跟 channel 关系最亲密的语句，它是被专门设计出来处理通道的，因为每个 case 后面跟的都是通道表达式，可以是读，也可以是写。

```
ch := make(chan int)
q := make(chan int)

go func(ch, q chan int) {
    for i := 0; i < 10; i++ {
        num := <-ch
        fmt.Println(num)
    }
    q <- 1
}(ch, q)

fibonacci := func(ch, q chan int) {
    x, y := 0, 1
    for {
        select {
        case ch <- x: // 写入
            x, y = y, x+y
            break // 你觉得是否会影响 for 语句的循环？
        case <-q: // 读取
            fmt.Println("quit")
            return
        }
    }
}
fibonacci(ch, q)
复制代码

```

上面的代码是利用 channel 实现的一个斐波拉契数列。select 还可以有 default 语句，该语句会在其它 case 都被阻塞的情况下执行。

关注的问题

1.  select 只要有默认语句，就不会被阻塞，换句话说，如果没有 default，然后 case 又都不能读或者写，则会被阻塞
    
2.  nil 的 channel，不管读写都会被阻塞
    
3.  select 不能够像 `for-range` 一样发现 channel 被关闭而终止执行，所以需要结合 `multi-valued assignment` 来处理
    
4.  如果同时有多个 case 满足了条件，会使用伪随机选择一个 case 来执行
    
5.  select 语句如果不配合 for 语句使用，只会对 case 表达式求值一次
    
6.  每次 select 语句的执行，是会扫码完所有的 case 后才确定如何执行，而不是说遇到合适的 case 就直接执行了。
    

总结
==

本文内容很简单易懂，希望大家彻底掌握了 channel 的使用。一切源码的研究都是为了更好的使用，后面的文章将开始研究 channel 的源码实现。

本文几个重要问题再次总结下，也是经常面试的常考点。

1.  向 close 的 channel 写数据、再次 close 都会触发 `runtime panic`。
    
2.  向 nil channel 写、读取数据，都会阻塞，可以利用这点来优化 for + select 的用法。
    
3.  channel 的关闭最好在写入方处理，读的协程不要去关闭 channel，可以通过单向通道来表明 channel 在该位置的功能。
    
4.  如果有多个写协程的 channel 需要关闭，可以使用额外的 channel 来标记，也可以使用 `sync.Once` 或者 `sync.Mutex` 来处理。
    
5.  channel 不管是读写都是并发安全的，不会出现多个协程同时读或者写的情况，从而实现了 CSP。
    

**参考资料**

*   [1] [Go Channel 详解](https://colobu.com/2016/04/14/Golang-Channels/)
    
*   [2] [The Nature Of Channels In Go](https://www.ardanlabs.com/blog/2014/02/the-nature-of-channels-in-go.html)
    

**下期预告**

channel 这些特性是如何在代码层面实现的？下期会先将 channel 内部两个重要的数据结构：循环队列实现的 buf，以及单向链表实现的协程读写队列。
