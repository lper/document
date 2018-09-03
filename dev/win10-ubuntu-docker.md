## 说明

   >     微软和Ubuntu深入合作推出了基于win桌面运行Ubuntu系统.
   >     
   >     为了满足开发同学也在win下也可以使用ubuntu的开发环境.
   >     
   >     通过利用win上的Linux子系统Ubuntu16.04能否安装docker并正常使用

    

## 首先在安装Ubuntu应用之前，我们要做一些事情，避免安装和使用过程中，遇到各种坑。
 
       
    

 - 到控制面板找到下图的window功能，勾选对应的两项内容点击确定，最后会提示你重启电脑，重启之后进行后面的操作
   ![win10 install linux][1]
   


    
 - 以下是安装Ubuntu应用操作相关图片 打开微软应用商店，搜索Ubuntu，选择16.04安装版本，这个版本测试是没有问题， 其它版本的没有测试。
    ![安装Ubuntu应用操作][2]



 - 安装完成后,要以管理员身份运行刚刚安装的Ubuntu应用，不然执行docker命令时会提示：cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running，然后就是进行Ubuntu系统初始化操作，等待一会儿，会提示你输入一个用户名和密码，按照提示输入就可以了。
     ![管理员身份运行刚刚安装的Ubuntu应用][3]
     
   - 更换软件源apt源
       > ` cd /etc/apt/`
       
       > `      sudo cp sources.list sources.list.bak && sudo vim sources.list`
       
       > `#deb包 `
       
       > ` deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse`
       
       > ` deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse`
       
       > ` deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse`
       
       > ` deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse`
       
       > `#测试版源 `
       
       > ` deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse`
       
       > `#源码 `
       
       > ` deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse`
       
       > ` deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse`  
       
       > ` deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse`  
       
       > ` deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse`
       
       > ` #测试版源 `  
       
       > ` deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse` 
       
       > `#Canonical 合作伙伴和附加 `
       
       > ` deb http://archive.canonical.com/ubuntu/ xenial partner` 
       
       > ` deb http://extras.ubuntu.com/ubuntu/ xenial main`  
            
   下面就是安装docker，直接贴官方的Ubuntu安装docker的教程，如果访问不了记得架梯子。https://docs.docker.com/install/linux/docker-ce/ubuntu/
   
   > `# step 1: 安装必要的一些系统工具`

   > `sudo apt-get update`
   > `sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common`

   > `# step 2: 安装GPG证书`

   > `curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`
   

   > `# Step 3: 写入软件源信息`

   > `sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"`


   > `# Step 4: 更新并安装 Docker-CE`

   > ` sudo apt-get update`
   > `sudo apt-get install docker-ce=17.03.2~ce-0~ubuntu-xenial`
   

   > `#安装指定版本的Docker-CE:`
   
   > `#Step 1: 查找Docker-CE的版本:`
   
   > `#apt-cache madison docker-ce`
   
   > `#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages`
   
   > `#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages` 
      
   > `# sudo apt-get -y install docker-ce=[VERSION]` 
       
   > `#   docker-ce | 17.03.0~ce-0~ubuntu-xenial |`    

  [1]: https://raw.githubusercontent.com/lper/document/master/dev/win-liunx.png
  [2]: https://raw.githubusercontent.com/lper/document/master/dev/win-ubuntu.png
  [3]: https://raw.githubusercontent.com/lper/document/master/dev/ubuntu-scop.png