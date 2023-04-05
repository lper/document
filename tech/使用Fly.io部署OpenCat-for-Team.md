title:: 使用Fly.io部署OpenCat for Team

-
- 因为国内无法直接访问openai，openai也不支持国内区域。部署后，可以让朋友们一起使用，免去了一些复杂过程。
-
- [OpenCat]([OpenCat on the App Store (apple.com)](https://apps.apple.com/us/app/opencat/id6445999201))
	- 一款ChatGPT客户端，支持mac和iOS
	- 支持docker`私有部署`，部署后可以分享给朋友使用
- [Fly.io](https://fly.io/)
	- [免费计划](https://fly.io/docs/about/pricing/#plans)可以使用3个共享cpu的虚拟机，3GB储存和160GB下行数据，足够支持我们部署后的使用
-
- 准备
	- 至少一个openai的key，多个可以轮询
	- [Fly.io](https://fly.io/)账号
	- 若要使用微软TTS功能，需要[Azure](https://azure.microsoft.com/zh-cn/)账号
-
- 私有部署
	- 使用[𝔽𝕣𝕠𝕤𝕥 𝕄𝕚𝕟𝕘](https://twitter.com/frostming90)分享的[配置](https://gist.github.com/frostming/05f41d3e35bf798fd224bc23fc07fcd6)
	  ```toml
	  kill_signal = "SIGINT"
	  kill_timeout = 5
	  primary_region = "sin"
	  processes = []
	  
	  [build]
	    image = "bayedev/opencatd"
	  
	  [mount]
	    source = "opencat"
	    destination = "/opt/db"
	  
	  [experimental]
	    auto_rollback = true
	  
	  [[services]]
	    internal_port = 80
	    protocol = "tcp"
	  
	    [services.concurrency]
	      hard_limit = 25
	      soft_limit = 20
	      type = "connections"
	  
	    [[services.ports]]
	      force_https = true
	      handlers = ["http"]
	      port = 80
	  
	    [[services.ports]]
	      handlers = ["tls", "http"]
	      port = 443
	  
	    [[services.tcp_checks]]
	      grace_period = "1s"
	      interval = "15s"
	      restart_limit = 0
	      timeout = "2s"
	  ```
	-
	- 安装fly的命令行工具[Install flyctl · Fly Docs](https://fly.io/docs/hands-on/install-flyctl/)
	  ```bash
	  #Mac
	  brew install flyctl
	  
	  #Linux
	  curl -L https://fly.io/install.sh | sh
	  
	  #Windows
	  powershell -Command "iwr https://fly.io/install.ps1 -useb | iex"
	  
	  ```
	- 登录fly
	  ```bash
	  fly auth login
	  ```
	- 本地建立目录，创建fly.tomal文件并复制上面的配置内容
	- 创建和部署App
	  >>创建的区域节点尽量选择美国的
	  
	  ```bash
	  fly launch --copy-config
	  fly vol create opencat --size 1
	  fly deploy
	  ```
	- 绑定域名
	- Fly.io提供SSL证书，在dashboard中配置一下
	- 在OpenCat中创建团队，填入绑上的域名，填上openai的key就能直接使用了
	- 若要使用微软的TTS，登录Azure后，创建[语音资源](https://portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices)，在OpenCat配置中选择区域，填上token