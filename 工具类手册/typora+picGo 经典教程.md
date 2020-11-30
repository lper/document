> typora+picGo 经典教程

### 简介：typora 有着 markdown 编辑器之王之称，它有着简约、优雅、实用等优点。然而图片的存储问题就限制了这个软件的使用，我们搭配使用 picGo 这个免费的图床软件就可以轻易解决这个问题了。并且搭配 gitee 码云，用上去，嗯，真香，丝滑！

一、 typora 安装
------------

#### 考虑到官网下载或许会有点慢，这里提供目前最新版本的 typora 的安装包，链接如下：[wwa.lanzous.com/iE60rehxw8j](https://wwa.lanzous.com/iE60rehxw8j)

#### 安装过程就是无脑的 next，路径选择好，不安装在 C 盘即可。

二、picGo 安装
----------

#### picGo 的安装包链接如下：[wwa.lanzous.com/iENWHehxwqh](https://wwa.lanzous.com/iENWHehxwqh)

三、node 安装
---------

#### 通过 npm，下载 picGo 插件，从而使用 gitee。

#### node 中文网：[nodejs.cn/download/](http://nodejs.cn/download/)

#### 在官网下载. msi 安装包，进行安装，修改安装位置后，无脑按 next 即可。

### 我们在 node 的文件夹新建两个文件夹 node_global、node_cache。

### 此时我们在 cmd 中输入 npm list -global:

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993b99e83edd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 在 cmd 中输入下面的两行代码：（根据你自己的位置进行调整代码）

#### npm config set prefix "D:\nodejs\node_global"

#### npm config set cache "D:\nodejs\node_cache"

#### 输入命令 npm list -global

#### 输入命令 npm config set registry=http://registry.npm.taobao.org 配置镜像站

#### 输入命令 npm config list 显示所有配置信息，我们关注一个配置文件:

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993b9d14887c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 检查一下镜像站 npm config get registry:

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993b9d29d3aa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 我们可以稍微测试一下，在 cmd 中输入 npm info vue:

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993b9f37503b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 我们输入代码 npm install -g:(-g 是指 global 文件夹)

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993ba01f319f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 此时再次查看 global 中有什么文件，global 目录下不再为空了：

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993ba0f1fdd4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 注意，此时，默认的模块 D:\nodejs\node_modules 目录，将会改变为 D:\nodejs\node_global\node_modules 目录，如果直接运行 npm install 等命令会报错的。我们需要做 1 件事情：增加环境变量 NODE_PATH 内容是：D:\nodejs\node_global\node_modules

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bc6b95f54?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 然后对 path 环境变量添加 D:\nodejs\node_global

#### 这个时候重启 picGo 软件，进行插件的安装：

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bc90141e9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

四、gitee 创建图片上传的专属仓库
-------------------

#### 按下列步骤进行操作：

#### 第一步：

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bc9a1cefd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 第二步：

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bc9ce4fb5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 第三步：创建私人令牌。

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bce11a6eb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 第四步：

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bcae58a9e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 此时切记，记住这个 token。

五、typora 搭配 picGo
-----------------

#### 打开 picGo

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bebf1d857?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 找到 githubPlus

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bf2cbca5b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 按确定，并设置为默认图床。

#### 打开 typora，找到偏好设置，进入图像，在进行如下操作：

![](https://user-gold-cdn.xitu.io/2020/7/11/1733993bf4bd8561?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 按下验证图片上传选项，如果出现了 success，则证明配置以成功。

### **接下来，你就可以 typora 愉快玩耍了，进行写博客、做笔记等相当舒适！！！**