---
layout:     post
title:      "记一次Linux下SSL证书安装"
subtitle:   "成功了..."
date:       2019-07-16 22:59:59
author:     "Qing"
header-img: "img/post-bg-2019-07-16.jpg"
catalog: true
tags: 
    - linux
    - ssl
    - nginx
---

> “Yeah It's success. ”


#### 证书申请
可以去[免费的证书平台](https://freessl.cn)

##### 第一步
申请证书后会有这些验证信息

![第一步](/img/in_pots/20190716/1.PNG "申请证书后会有这些验证信息")

##### 第二步
去阿里云的DNS解析平台添加对应的解析信息，类型要选择TXT类型。
![第二步](/img/in_pots/20190716/2.PNG "ss")


##### 第三步
点击验证，这里不要点太快会出现繁忙稍后再试
![第三步](/img/in_pots/20190716/3.PNG "点击验证，这里不要点太快会出现繁忙稍后再试")


以上三步完成后，证书就下载到你本地了。


#### 证书上传服务器
将下载的证书内的 ``.key``  ``.pem`` 的文件放在服务器端自己定义的文件夹或者对应中间件文件夹内部

我尝试的是将ssl证书使用nginx进行配置，``nginx.conf``，配置如下：

```


user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;


    include /etc/nginx/conf.d/*.conf;

    #增加ssl配置：
    server {
        listen       443 ssl;
        server_name  http://freezone.fun;
        root         /usr/share/nginx/html;

	   ssl on;
	   ssl_certificate full_chain.pem;
       ssl_certificate_key private.key;
       ssl_session_timeout 5m;
       ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	   ssl_ciphers ECDH:AESGCM:HIGH:!RC4:!DH:!MD5:!aNULL:!eNULL;
	   ssl_prefer_server_ciphers on;


        include /etc/nginx/default.d/*.conf;

        location /tool {
         proxy_pass http://127.0.0.1:8080;
         proxy_redirect default;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

    #同时把80端口的重定向到https
    server{
    listen       80;
        server_name  localhost;

        if ($host = "freezone.fun"){
                return 302 https://freezone.fun$request_uri;
        }
        return 302 https://$host$request_uri;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
        proxy_pass http://freezone.fun;
        proxy_redirect default;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }



}


```

然后重启nginx 配置生效


```
service nginx restart
```







