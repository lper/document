> 使用 Brotli 提高网站访问速度

在优化网站打开速度上，我们有很多的方法，而其中一个就是减少诸如 Javascript 和 CSS 等资源文件的大小，而减少文件大小的方法除了在代码上下功夫外，最常用的方法就是使用压缩算法对文件进行压缩。

目前，网站普遍使用的是`gzip`压缩算法，当然你可能还知道`deflate`和`sdch`算法，但是最近两年新兴了一个新的压缩算法：[Brotli](https://en.wikipedia.org/wiki/Brotli)，下面我将会对这个算法进行简单的介绍。

### 什么是 Brotli

Brotli 最初发布于 2015 年，用于网络字体的离线压缩。Google 软件工程师在 2015 年 9 月发布了包含通用无损数据压缩的 Brotli 增强版本，特别侧重于 HTTP 压缩。其中的编码器被部分改写以提高压缩比，编码器和解码器都提高了速度，流式 API 已被改进，增加更多压缩质量级别。新版本还展现了跨平台的性能改进，以及减少解码所需的内存。

与常见的通用压缩算法不同，Brotli 使用一个预定义的 120 千字节字典。该字典包含超过 13000 个常用单词、短语和其他子字符串，这些来自一个文本和 HTML 文档的大型语料库。预定义的算法可以提升较小文件的压缩密度。

使用 brotli 取代 deflate 来对文本文件压缩通常可以增加 20% 的压缩密度，而压缩与解压缩速度则大致不变。

### 浏览器支持情况

![](https://raw.sevencdn.com/lper/document/master/img/196812783-5913d017054ae_articlex.png)

*   Chrome 从版本 49 开始支持，但是完整的支持是在版本 50（2016 年 5 月 27 日开始支持）。
    
*   Firefox 从版本 52 开始支持。
    
*   IE 全版本不支持，但是 Edge 从版本 15 开始支持。
    
*   Safari 全系不支持。
    
*   Opera 从版本 44 开始支持。
    

支持 Brotli 压缩算法的浏览器使用的内容编码类型为`br`，例如以下是 Chrome 浏览器请求头里`Accept-Encoding`的值：

```
Accept-Encoding: gzip, deflate, sdch, br
```

如果服务端支持 Brotli 算法，则会返回以下的响应头：

```
Content-Encoding: br
```

> 需要注意的是，只有在 HTTPS 的情况下，浏览器才会发送`br`这个 Accept-Encoding。

### 关于性能

下面是 LinkedIn 做的一个性能测试结果：

![](https://raw.sevencdn.com/lper/document/master/img/1954126711-5916930f96151_articlex.png)

<table><thead><tr><th>Algorithm</th><th>Quality</th><th>Compression Time (ms)</th><th>Decompression Time (ms)</th></tr></thead><tbody><tr><td>gzip</td><td>6</td><td>169</td><td>35</td></tr><tr><td>gzip</td><td>9</td><td>284</td><td>27</td></tr><tr><td>zopfli</td><td>15</td><td>37,847</td><td>32</td></tr><tr><td>zopfli</td><td>100</td><td>194,460</td><td>38</td></tr><tr><td>zopfli</td><td>1000</td><td>1,855,480</td><td>29</td></tr><tr><td>brotli</td><td>4</td><td>109</td><td>24</td></tr><tr><td>brotli</td><td>5</td><td>193</td><td>20</td></tr><tr><td>brotli</td><td>5</td><td>517</td><td>23</td></tr><tr><td>brotli</td><td>11</td><td>11,913</td><td>22</td></tr></tbody></table>

可以看到，Brotli 的压缩率更高，意味着通过 Brotli 算法压缩的文件，文件大小更小，但是由表格可以看到，Brotli 的压缩时间比 gzip 要多，而解压时间则相当。所以在运行中（on-the-fly）使用 Brotli 算法压缩文件可能并不是一个很好的方案，下面我们再探讨下。

更多的评测可以看以下两个链接：

*   [](https://cran.r-project.org/web/packages/brotli/vignettes/benchmarks.html)[https://cran.r-project.org/we...](https://cran.r-project.org/web/packages/brotli/vignettes/benchmarks.html)
    
*   [](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)[https://hacks.mozilla.org/201...](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
    

### 使用 Brotli

Brotli 有更高的压缩率，但是同时也需要更长的压缩时间，所以在请求的时候实时进行压缩并不是一个很好的实践（当然你可以这么做）。我们可以预先对静态文件进行压缩，然后直接提供给客户端，这样我们就避免了 Brotli 压缩效率低的问题，同时使用这个方式，我们可以使用压缩质量最高的等级去压缩文件，最大程度的去减小文件的大小。

另外，由于不是所有浏览器都支持 Brotli 算法，所以在服务端，我们需要同时提供两种文件，一个是经过 Brotli 压缩的文件，一个是原始文件，在浏览器不支持 Brotli 的情况下，我们可以使用 gzip 去压缩原始文件提供给客户端。

具体的实现可以参照下 Linkin 的这篇文章：[](https://engineering.linkedin.com/blog/2017/05/boosting-site-speed-using-brotli-compression)[https://engineering.linkedin....](https://engineering.linkedin.com/blog/2017/05/boosting-site-speed-using-brotli-compression)。

### 在 Nginx 上启用 Brotli

nginx 目前并不支持 Brotli 算法，需要使用第三方模块，例如`ngx_brotli`进行实现。下面是简单的安装步骤。

#### 安装及配置

下载`ngx_brotli`模块及其依赖：

```
$ git clone https://github.com/google/ngx_brotli
$ cd ngx_brotli
$ git submodule update --init
```

编译 Nginx 时加入`ngx_brotli`模块：

```
$ cd /path/to/nginx_source/
$ ./configure --add-module=/path/to/ngx_brotli
$ make && make install
```

在 Nginx 配置文件的`http`块下增加以下指令：

```
brotli               on;  
brotli_comp_level    6;  
brotli_buffers       16 8k;  
brotli_min_length    20;  
brotli_types         *;
```

以上是`on-the-fly`的配置方式，如果是要响应已经使用 Brotli 压缩过的文件，则使用`brotli_static`指令。下面是`ngx_brotli`模块相关指令的一些简单解析。

#### 模块指令解析

##### brotli_static

启用后将会检查是否存在带有`br`扩展的预先压缩过的文件。如果值为`always`，则总是使用压缩过的文件，而不判断浏览器是否支持。

##### brotli

是否启用在 on-the-fly 方式压缩文件，启用后，将会在响应时对文件进行压缩并返回。

##### brotli_types

指定对哪些内容编码类型进行压缩。`text/html`内容总是会被进行压缩。

##### brotli_buffers

设置缓冲的数量和大小。大小默认为一个内存页的大小，也就是`4k`或者`8k`。

##### brotli_comp_level

设置压缩质量等级。取值范围是 0 到 11.

##### brotli_window

设置窗口大小。

##### brotli_min_length

设置需要进行压缩的最小响应大小。

> 具体信息请参看：[](https://github.com/google/ngx_brotli)[https://github.com/google/ngx...](https://github.com/google/ngx_brotli)

### 参考

*   [](https://en.wikipedia.org/wiki/Brotli)[https://en.wikipedia.org/wiki...](https://en.wikipedia.org/wiki/Brotli)
    
*   [](https://engineering.linkedin.com/blog/2017/05/boosting-site-speed-using-brotli-compression)[https://engineering.linkedin....](https://engineering.linkedin.com/blog/2017/05/boosting-site-speed-using-brotli-compression)
    
*   [](https://github.com/google/ngx_brotli)[https://github.com/google/ngx...](https://github.com/google/ngx_brotli)
    
*   [](http://caniuse.com/#search=brotli)[http://caniuse.com/#search=br...](http://caniuse.com/#search=brotli)