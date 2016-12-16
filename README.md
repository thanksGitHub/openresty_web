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

##根据openresty的一些特性进行对Nginx的监视处理
![MacDown logo](https://cloud.githubusercontent.com/assets/2137369/15272097/77d1c09e-1a37-11e6-97ef-d9767035fc3e.png)

根据上图，如果我们要对work进行监控的话，我们就可以在__init_work_by_lua*__阶段进行各种操作。如果只是对心跳检测建议可以是用_ngx.socket.udp_,如果是对一些信息做一些交换，建议使用_resty-http_模块，如果是对配置信息定时做一些调整，有_resty-redis_，或_resty-memcache_等。

    local http   = require "resty.http.simple"
	local new_timer = ngx.timer.at
  	function request()
     local res, err = http.request("204.236.225.73", 80, {
      headers = { Cookie = "foo=bar"} })
      if not res then
         ngx.log(ngx.ERR, "http failure: ", err)
        return
      end

      if res.status >= 200 and res.status < 300 then
        ngx.log(ngx.ERR,"My IP is: " .. res.body)
      else
        ngx.log(ngx.ERR,"Query returned a non-200 response: " .. res.status)
      end
  	end

  	local function get_task()
      local res, err =new_timer(0, request)
      if not res then
         ngx.log(ngx.ERR, "http failure:")
        return
      end
      -- new_timer(3,get_task)
  	end
 	get_task()

##balance_by_lua对upstream进行动态更换，在不重启的情况下

####开始实践：

	upstream backend {
        server 127.0.0.1:8080;
          balancer_by_lua_block {
            local balancer = require "ngx.balancer"
            local host = "127.0.0.1"
            local port = 10000
            //在这个之间可以设置各种负载算法，而且，还可以使用共享内存的方式的进行动态修改
            local ok, err = balancer.set_current_peer(host, port)
            if not ok then
                ngx.log(ngx.ERR, "failed to set the current peer: ", err)
               return ngx.exit(500)
            end
        }

    }
    server {
        listen 8090;
        location / {
	      proxy_pass http://backend;
        }
	}

   	server {
        listen 10000;
        location / {
            echo "this is a faker";
        }
    }


