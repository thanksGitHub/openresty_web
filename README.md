#openresty_web
这是一个内部分享使用openresty搭建的一个简单web服务器主要目的还是为了展示openresty的一些用法

##快速实践一,节约成本的前提的下多个域名使用同一个ip
原理:根据请求中带有的hostname进行判断将不同域名映射到不同的location中,当然这个很多时候就是做实验使用的  
####具体代码
         local name=ngx.var.host  
         if name=="xxx.xxxxxx.xxx" then  
             return ngx.redirect("/index.html");  
         else  
             ngx.exec("/logs");  
          end  
##快速实践二,快速搭建一个https网站(单域名的情况下)
+ 申请证书  
+ 上传证书到nginx的目录下  
+ 开启https  

> __修改nginx.conf文件__  
   listen 443;  
   ssl on;    
   ssl_certificate key/1_mobile.zzz.zzzz.zz.crt；   
   ssl_certificate_key key/zzz.xxx.ccc.key;  

##快速实践三,对线上访问日志传送到指定的日志收集中
对于这种描述的情况，我们首先应该了解的日志的一些格式，互联网中我们可以收集很多这方面的资料。我就简单的介绍一下代码的写法。

	openresty 提供了非阻塞的cosocket，可以用来实现tcp/或者是udp协议。
	local sock = ngx.socket.tcp()
 	local ok, err = sock:connect(...)
 	if not ok then
     return nil, err
 	end
 	return sock
 	sock:send("日志信息")

