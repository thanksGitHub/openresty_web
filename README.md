#openresty_web
这是一个内部分享使用openresty搭建的一个简单web服务器主要目的还是为了展示openresty的一些用法


##快速实践一,节约成本的前提的下多个域名使用同一个ip
原理:根据请求中带有的hostname进行判断将不同域名映射到不同的location中,当然这个很多时候就是做实验使用的    
         local name=ngx.var.host  
         if name=="mobile.yidaren.top" then  
             return ngx.redirect("/index.html");  
         else  
             ngx.exec("/logs");  
          end  
##快速实践二,快速搭建一个https网站(单域名的情况下)
+ 申请证书  
+ 上传证书到nginx的目录下  
+ 开启https  
         # listen       443;  
         # ssl on;  
         # ssl_certificate key/1_mobile.yidaren.top_bundle.crt;  
         # ssl_certificate_key key/mobile.yidaren.top.key;  
      
