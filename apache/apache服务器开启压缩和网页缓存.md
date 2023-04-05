> *   windows下apache服务器开启压缩和网页缓存
    ==========================
    
    找到配置文件：http.conf
    
    **apache开启压缩**
    
    一、开启配置，去除下面代码前面的#号  
    LoadModule deflate_module modules/mod_deflate.so  
    LoadModule headers_module modules/mod_headers.so
    
    二、添加压缩规则
    
    <IfModule deflate_module>  
    #必须的，就像一个开关一样，告诉apache对传输到浏览器的内容进行压缩  
    SetOutputFilter DEFLATE  
      
    #压缩级别，1-9，9为最高  
    DeflateCompressionLevel 6  
      
    #不进行压缩的文件  
    SetEnvIfNoCase Request_URI .(?:gif|jpe?g|png)$ no-gzip dont-vary #设置不对后缀gif，jpg，jpeg，png的图片文件进行压缩  
    SetEnvIfNoCase Request_URI .(?:exe|t?gz|zip|bz2|sit|rar)$ no-gzip dont-vary #同上，就是设置不对exe，tgz，gz。。。的文件进行压缩  
    SetEnvIfNoCase Request_URI .(?:pdf|mov|avi|mp3|mp4|rm)$ no-gzip dont-vary  
      
    #针对代理服务器的设置  
    <IfModule headers_moudle>  
    Header append vary User-Agent  
    </IfModule>  
    </IfModule>
    
    **apache开启网页缓存**
    
    一、开启配置，去除下面代码前面的#号
    
    LoadModule expires_module modules/mod_expires.so
    
    二、添加缓存规则
    
    <IfModule expires_module>  
    ExpiresActive On  
    ExpiresDefault "access plus 1 month"  
    ExpiresByType text/html "access plus 1 months"  
    ExpiresByType text/css "access plus 1 months"  
    ExpiresByType image/gif "access plus 1 months"  
    ExpiresByType image/jpeg "access plus 1 months"  
    ExpiresByType image/jpg "access plus 1 months"  
    ExpiresByType image/png "access plus 1 months"  
    EXpiresByType application/x-shockwave-flash "access plus 1 months"  
    EXpiresByType application/x-javascript "access plus 1 months"  
    ExpiresByType video/x-flv "access plus 1 months"  
    </IfModule>
    
*   **相关阅读:**  
    
*   原文地址：https://www.cnblogs.com/bubuchu/p/6078555.html