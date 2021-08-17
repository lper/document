> window 中关于端口被占用的解决 443 failed: port is already allocat.
------------------------------------------------------

背景
--

当前遇到的问题

> Error starting userland proxy: Bind for 0.0.0.0:443 failed: port is  
> already allocated

解决
--

### 智能推荐

 [![][img-0] 

### [Error：That port is already in use](https://www.pianshen.com/article/44981939382/ "Error：That port is already in use")

运行 django 项目发现端口被占用了 ，我猜是上次运行时，直接关闭了 vscode，但是端口还占用呢？ 使用 lsof -i：8000 查看了占用此端口的进程再杀死。之前还想杀死端口，我真是傻了。黄色框为错误，红色为正确，绿色为正确。 参考：https://www.cnblogs.com/xuepangzi/p/11104329.html...

### 猜你喜欢

 [![][img-1] 

### [入门小白想学渲染背景颜色，听听大神是如何用 maya 软件渲染的](https://www.pianshen.com/article/51051801787/ "入门小白想学渲染背景颜色，听听大神是如何用maya软件渲染的")

MAYA 中的渲染背景默认情况下都是黑色的，如果我们想改变成自己需要的颜色怎么办呢? 今天为大家分享如何改变它的渲染背景颜色。（如果想更多了解游戏建模可以加小编游戏建模企鹅交流社团：1046777540，还可以领取免费的教程哦） 1、打开 Maya, 点击渲染按钮。我们看到默认的是黑色渲染背景。 2、点击视图窗左上角的第二个按钮：Camera Attribute ，如图，弹出了摄像机的属性编辑器。 3、拖...

 [![][img-2] 

### [Visual SVN 的安装](https://www.pianshen.com/article/2692563431/ "Visual SVN的安装")

Visual SVN 的 安装 作为一个程序开发人员，就算自己一个人写程序，也应该有一个 SVN 版 本控制系统，以便对开发代码进行有效的管理。今天我就介绍一个在 Windows 环境下简单快速搭建 SVN 服 务器的方法。 通常的 SVN 服 务器是搭建在 Linux 等 系统下，例如用 Apache+SVN 配置， Linux 下的 SVN 性 能会非常好，但配置有些繁琐，如果 SVN 服务器...

 [![][img-3] 

### [冒泡排序算法详解](https://www.pianshen.com/article/909998948/ "冒泡排序算法详解")

本文关于冒泡算法的介绍是参考：经典排序算法（1）——冒泡排序算法详解 这篇文章的，该文章也有讲述如何实现冒泡排序的。 冒泡排序 (Bubble Sort) 是一种典型的交换排序算法，通过交换元素的位置进行排序。 算法的基本思想 冒泡排序的基本思想是：从无序序列头部开始，进行两两比较，根据大小交换位置，知道最后将最大(小) 的数据交换到了无序队列的队尾，从而成为了有序序列的一部分...

 [![][img-4] 

### [工作忙成狗，如何一年搞定 100 本书，打败 99% 的同龄人？](https://www.pianshen.com/article/3254770112/ "工作忙成狗，如何一年搞定 100 本书，打败 99% 的同龄人？")

这篇文章来自我的另一个公众号「价值前瞻」，更多内容，欢迎关注。 不少同学每年都有阅读的小目标，又到一年年底了，当初的 flag 还好么？ 工作忙成狗， 如何一年搞定 100 本书， 打败 99% 的同龄人？ 各位朋友早上好，我是 Lemonbit，今天想跟大家聊聊书籍阅读这件事。 有人一年阅读了 100 本书，有人一年阅读数量不到 10 本，同样都是一年，难道真的是时间的问题，还是阅读方法本身有问...

[img-0]:data:text/html;base64,NDA0

[img-1]:data:text/html;base64,NDA0

[img-2]:data:text/html;base64,NDA0

[img-3]:data:text/html;base64,NDA0

[img-4]:data:text/html;base64,NDA0