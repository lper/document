> Golang 学习——基于 Gin 框架进行 httptest 单元测试

### 基于 Gin 框架进行 httptest 单元测试

*   [一. 实例代码](#_7)
    
    *   [1. 全局变量及 main 函数](#1main_13)
        
    *   [2. 初始化路由](#2_28)
        
    *   [3. 三个主要功能](#3_46)
        
        *   [3.1 首页](#31__48)
            
        *   [3.2 导入用户](#32__54)
            
        *   [3.3 抽奖](#33__77)
            
        
    
*   [二. 测试工具函数](#_105)
    
*   [2.1 ParseToStr 将 map 中的键值对输出成 querystring 形式](#21__ParseToStr_mapquerystring_108)
    
*   [2.2 Get 根据特定请求 uri，发起 get 请求返回响应](#22__Get_uriget_121)
    
*   [2.3 ParseToStr 将 map 中的键值对输出成 querystring 形式](#23__ParseToStr_mapquerystring_134)
    
*   [2.4 PostJson 根据特定请求 uri 和参数 param，以 Json 形式传递参数，发起 post 请求返回响应](#24__PostJson_uriparamJsonpost_156)
    
*   [三. 开始进行 httptest 测试](#_httptest__172)
    
*   [四. 运行单元测试，查看结果](#_224)
    
*   [五. 总结](#_229)
    

昨天晚上在学习慕课网的课程时，写了个简单的抽奖 demo，打算简单测试在并发场景下临界资源是否被修改的问题。

然后前后折腾了好久才测试成功，记录下自己在进行`httptest`单元测试时学到的知识。

一. 实例代码
=======

以下代码是要的测试内容，大致有三个功能：

*   index 首页，GET 请求
    
*   导入抽奖用户，POST 请求
    
*   抽奖，GET 请求
    

1. 全局变量及 main 函数
----------------

记得初始化锁，否则不起作用。

```
// 用户列表 共享变量（临界资源）
var userList []string
// gin引擎
var router *gin.Engine
// 互斥锁
var mux sync.Mutex

func main() {
 mux = sync.Mutex{} // 初始化锁
 router.Run(":8080")
}
复制代码

```

2. 初始化路由
--------

主要初始化了三个功能的路由

```
func init() {
 router = gin.Default()
 // 路由组
 userGroup := router.Group("/user")
 {
  // 首页
  userGroup.GET("/index", Index)
  // 导入用户
  userGroup.POST("/import", ImportUsers)
  // 抽奖
  userGroup.GET("/lucky", GetLuckyUser)
 }
}
复制代码

```

3. 三个主要功能
---------

请求成功后，每个页面都是返回一个字符串（包含各自的信息）

### 3.1 首页

```
func Index(c *gin.Context) {
 c.String(http.StatusOK, "当前参与抽奖的用户人数:%d", len(userList))
}
复制代码

```

### 3.2 导入用户

```
func ImportUsers(c *gin.Context) {
 strUsers := c.Query("users")
 users := strings.Split(strUsers, ",")
 // 在操作 全局变量 userList 之前加互斥锁，加完锁记得释放
 mux.Lock()
 defer mux.Unlock()
 // 统计当前已经在参加抽奖的用户数量
 currUserCount := len(userList)

 // 将页面提交的用户导入到 userList 中，参与抽奖
 for _, user := range users {
  user = strings.TrimSpace(user)
  if len(user) > 0 {
   userList = append(userList, user)
  }
 }
 // 统计当前总共参加抽奖人数
 userTotal := len(userList)
 c.String(http.StatusOK, "当前参与抽奖的用户数量:%d,导入的用户数量:%d", userTotal, (userTotal - currUserCount))
}
复制代码

```

### 3.3 抽奖

```
func GetLuckyUser(c *gin.Context) {
 var user string
 // 在操作 全局变量 userList 之前加互斥锁，加完锁记得释放
 mux.Lock()
 defer mux.Unlock()
 
 count := len(userList)
 if count > 1 {
  
  seed := time.Now().UnixNano()
  // 以随机数设置中奖用户, [0,count)中的随机值
  lottery_index := rand.New(rand.NewSource(seed)).Int31n(int32(count))
  user = userList[lottery_index]
  // 当前参与抽奖用户减 1
  userList = append(userList[0:lottery_index], userList[lottery_index+1:]...)
  c.String(http.StatusOK, "中奖用户为:%s，剩余用户数:%d", user, count-1)
  
 } else if count == 1 {
  user = userList[0]
  userList = userList[0:0] // 清空参与抽奖的用户列表
  c.String(http.StatusOK, "中奖用户为:%s，剩余用户数:%d", user, count-1)
 } else {
  c.String(http.StatusOK, "当前无参与抽奖的用户,请导入新的用户。")
 }
}
复制代码

```

二. 测试工具函数
=========

在`httptestUtil.go`文件中主要封装了以下工具函数：

### 2.1 ParseToStr 将 map 中的键值对输出成 querystring 形式

```
// ParseToStr 将map中的键值对输出成querystring形式
func ParseToStr(mp map[string]string) string {
 values := ""
 for key, val := range mp {
  values += "&" + key + "=" + val
 }
 temp := values[1:]
 values = "?" + temp
 return values
}
复制代码

```

### 2.2 Get 根据特定请求 uri，发起 get 请求返回响应

```
func Get(uri string, router *gin.Engine) *httptest.ResponseRecorder {
 // 构造get请求
 req := httptest.NewRequest("GET", uri, nil)
 // 初始化响应
 w := httptest.NewRecorder()

 // 调用相应的handler接口
 router.ServeHTTP(w, req)
 return w
}
复制代码

```

### 2.3 ParseToStr 将 map 中的键值对输出成 querystring 形式

构造 POST 请求，表单数据以 `querystring` 的形式加在 uri 之后

注意：form 表单的参数可以通过 `querystring` 的形式附在 URI 地址后面进行传递

这种方式，POST 请求获取参数是时要调用 `c.Query("users")`，而不是`c.PostFprm("users")`，更不是`c.Param("users)`

当然直接使用 `c.ShouldBind()` ，让 gin 自动判断是哪种方式的请求参数。  
代码如下：

```
// PostForm 根据特定请求uri和参数param，以表单形式传递参数，发起post请求返回响应
func PostForm(uri string, param map[string]string, router *gin.Engine) *httptest.ResponseRecorder {
 req := httptest.NewRequest("POST", uri+ParseToStr(param), nil)
 // 初始化响应
 w := httptest.NewRecorder()
 // 调用相应handler接口
 router.ServeHTTP(w, req)
 return w
}
复制代码

```

### 2.4 PostJson 根据特定请求 uri 和参数 param，以 Json 形式传递参数，发起 post 请求返回响应

```
// PostJson 根据特定请求uri和参数param，以Json形式传递参数，发起post请求返回响应
func PostJson(uri string, param map[string]interface{}, router *gin.Engine) *httptest.ResponseRecorder {
 // 将参数转化为json比特流
 jsonByte, _ := json.Marshal(param)
 // 构造post请求，json数据以请求body的形式传递
 req := httptest.NewRequest("POST", uri, bytes.NewReader(jsonByte))
 // 初始化响应
 w := httptest.NewRecorder()
 // 调用相应的handler接口
 router.ServeHTTP(w, req)
 return w
}
复制代码

```

三. 开始进行 httptest 测试
===================

**Golang** 规范是推荐一个方法写一个测试函数，并且以`Test`开头，后面跟方面名。

为了测试代码是否并发安全，就将三个功能的测试都写在同一个测试函数里，于是就命名为了`TestMVC`。

```
func TestMVC(t *testing.T) {
 var w *httptest.ResponseRecorder
 assert := assert.New(t)
 
 // 1.测试 index 请求
 urlIndex := "/user/index"
 w = Get(urlIndex, router)
 assert.Equal(200, w.Code)
 assert.Equal("当前参与抽奖的用户人数:0", w.Body.String())

 // 2.测试 import 请求，导入用户数
 var wg sync.WaitGroup // 定义wg, 用来阻塞 goroutine
 for i := 0; i < 100000; i++ {

  // 开一个等待
  wg.Add(1)
  go func(i int) { // i 不属于临界资源，是安全的
   defer wg.Done() // 一个 goroutine 跑完后要减1，

   // 测试 /user/import 请求，模拟从 form 表单中获取数据
   param := make(map[string]string)
   param["users"] = "user" + strconv.Itoa(i)
   urlImport := "/user/import"
   w = PostForm(urlImport, param, router)
   assert.Equal(200, w.Code)

  }(i)
 }
 // 等待上面的协程运行完，再接着测试
 wg.Wait()
 // 3.测试 urlIndex 请求，查看当前参与抽奖用户是否为 for 循环总数
 w = Get(urlIndex, router)
 assert.Equal(200, w.Code)
 assert.Equal("当前参与抽奖的用户人数:100000", w.Body.String())

 // 4.测试 抽奖
 urlLucky := "/user/lucky"
 w = Get(urlLucky, router)
 assert.Equal(200, w.Code)

 // 5.抽奖一次之后，再发起 index 请求，查看查看当前参与抽奖用户是否减 1
 w = Get(urlIndex, router)
 assert.Equal(200, w.Code)
 assert.Equal("当前参与抽奖的用户人数:99999", w.Body.String())
}
复制代码

```

四. 运行单元测试，查看结果
==============

运行结果如图：  
![][img-0]  
如图：  
在我个人电脑上，测试运行耗时：9.21s；根据 users 字段的名字也说明了执行了 100000 次，因为是并发执行的，所以顺序肯定不是从 1 到 100000 按序显示的（谁抢到 CPU 资源谁执行）

五. 总结
=====

从昨天晚上 7 点开始练习项目，进行单元测试，中间睡了 6 个小时吧。早上起来后，经过昨晚测试的磨练和学习，上午思路很清晰，不仅单元测试成功了，还将之前自己鼓捣的测试代码进行了重构和优化，直到今天上午 11 点多才正式完成。

第一次写 **Golang** 的 httptest 单元测试，整个过程就是边搜边学边实践，最后总算成功了。写一下 httptest 测试心得吧:

1.  在测试之前，封装好 get put 等请求的方法，封装到 httptestUtil，方便测试
    
2.  灵活应用测试框架，比如`Testify`，能少写很多 if 判断，(主要用来判断响应码和响应实体)。刚开始我就是`if else`写了很多判断，后来学了这个测试框架
    
3.  测试代码尽量简洁，保证可读性和可维护性。否则写一坨代码，容易逻辑混乱，而且看上去很烦，影响测试心智和测试准确性
    
4.  如果遇到新的测试问题，尽量多搜多查多静下来想一想，不要一股脑埋进去死挖问题原因。很可能你所纠结的问题并不是真正的原因
    
5.  如果测试顺利，那一切都好；如果测试不顺利，期间搜了很多资料，花费了大量时间进行测试，那么最后一定要写博客 (或笔记)，记录所学所想所得，否则以后还会遇到类似的问题
    
6.  测试代码最好贴到博客（或笔记 APP）上，方便以后查看
    
7.  最重要的一点，思路要清晰。测试很容易让人头大，烦躁，不要死磕，不妨停下来缓一缓，休息一下，让大脑放松下来
    

参考资料：  
1.[Gin 官方测试文档](https://gin-gonic.com/zh-cn/docs/testing/)  
2. [基于 golang gin 框架的单元测试](http://www.360doc.com/content/18/0718/23/9200790_771524922.shtml)  
3. [用 Testify 来改善 GO 测试和模拟](https://studygolang.com/articles/16799)
