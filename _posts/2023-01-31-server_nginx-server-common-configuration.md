---
title: Nginx服务器server节点常用配置
date: 2023-01-31 00:16:00 +0800
categories: [服务器]
tags: []
pin: false
---

Nginx服务器的server节点通常用来定义一个服务，Nginx服务器可以配置多个server节点，一个server通常用来定义一个单独项目（网站），也可以用一个 server来定义Nginx全局项目（网站），接下来我们总结Nginx服务器server节点的常用配置参数。

## 一、基础知识

### 1.1 常规配置

```conf
server {
    listen      80;
    server_name jkdev.cn www.jkdev.cn;
    root        /usr/share/nginx/html;
    index       index.php index.html index.htm;
}
```

- listen：监听端口
- server_name：域名
- root：项目路径
- index：默认访问文件

### 1.2 https配置

```conf
server {
    listen      443 ssl;
    server_name jkdev.cn www.jkdev.cn;
    root        /usr/share/nginx/html;
    index       index.php index.html index.htm;
    
    ssl_certificate   /etc/ssl/jkdev.cn/cert.pem;
    ssl_certificate_key  /etc/ssl/jkdev.cn/key.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
}
```

- ssl_certificate：证书公钥文件
- ssl_certificate_key：证书私钥文件
- ssl_session_timeout：缓存有效期
- ssl_protocols：安全链接可选的加密协议
- ssl_ciphers：加密算法
- ssl_prefer_server_ciphers on：使用服务器端的首选算法

### 1.3 从定向

如果访问域名不是www.jkdev.cn，强制从定向到<http://www.jkdev.cn>，并携带参数。permanent表示返回301永久重定向，地址栏显示重定向后的url

```conf
server {
    listen      80;
    server_name jkdev.cn www.jkdev.cn;
    root        /usr/share/nginx/html/php/www/public;
    index       index.php index.html index.htm;
   
    if ( $host != 'www.jkdev.cn' ) {
        rewrite ^(.*)$ https://www.jkdev.cn$1 permanent;
    }
}
```

在项目的根节点之下，如果访问文件为空，则重定向到根节点下的index.php文件。last表示url重写后，马上发起一个新的请求，再次进入server块，重试location匹配，超过10次匹配不到报500错误，地址栏url不变

```conf
server {
    listen      80;
    server_name jkdev.cn www.jkdev.cn;
    root        /usr/share/nginx/html/php/www/public;
    index       index.php index.html index.htm;
   
    if ( $host != 'www.jkdev.cn' ) {
        rewrite ^(.*)$ http://www.jkdev.cn$1 permanent;
    }

    location / {
        if (!-e $request_filename) {
        rewrite  ^(.*)$  /index.php/$1  last;
        }
    }
}
```

- localtion：项目下的路径

### 1.4 定义错误界面

当发生500、502、503、504这几种错误时，返回/50x.html。location部分定义了50x.html的访问位置，以保证找到自定义的50x页面

```conf
server {
    listen      80;
    server_name jkdev.cn www.jkdev.cn;
    root        /usr/share/nginx/html/php/www/public;
    index       index.php index.html index.htm;
   
    if ( $host != 'www.jkdev.cn' ) {
        rewrite ^(.*)$ https://www.jkdev.cn$1 permanent;
    }

    location / {
        if (!-e $request_filename) {
           rewrite  ^(.*)$  /index.php?s=/$1  last;
        }
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 1.5 fastcgi配置

当访问php文件时，转发给127.0.0.1:9000

```conf
server {
    listen      80;
    server_name jkdev.cn www.jkdev.cn;
    root        /var/www/html;
    index       index.php index.html index.htm;
   
    if ( $host != 'www.jkdev.cn' ) {
        rewrite ^(.*)$ https://www.jkdev.cn$1 permanent;
    }

    location / {
        if (!-e $request_filename) {
          rewrite  ^(.*)$  /index.php?s=/$1  last;
        }
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    
    location ~ \.php(.*)$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /var/www/html/$fastcgi_script_name;
        include        fastcgi_params;
        fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
    }
}
```

### 1.6 反向代理-端口转发

```conf
server {
    listen       443 ssl;
    server_name api.jkdev.cn;
   
    ssl_certificate   /etc/ssl/jkdev.cn/cert.pem;
    ssl_certificate_key  /etc/ssl/jkdev.cn/key.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    
    location /v2/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    } 
}
```

- proxy_pass：代理地址
- proxy_set_header：设置代理请求头

## 二、实际应用

根据1中的基础介绍，我们对Nginx的server配置参数有了基本了解，接下来我们模拟实际场景，编写Nginx的server配置

### 2.1 将所有http请求重定向到https

```conf
server {
    listen 80;
    rewrite ^(.*) https://$host permanent;
}
```

### 2.2 部署php博客（如wordpress、typecho）

```conf
server {
    listen       443 ssl;
    server_name blog.jkdev.cn;
    root        /usr/share/nginx/html/php/blog;
    index       index.php index.html index.htm;
   
    ssl_certificate   /etc/ssl/jkdev.cn/cert.pem;
    ssl_certificate_key  /etc/ssl/jkdev.cn/key.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    location / {
        if (!-e $request_filename) {
           rewrite  ^(.*)$  /index.php?s=/$1  last;
        }
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php(.*)$ {
        fastcgi_pass   php-fpm:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /var/www/html/blog/$fastcgi_script_name;
        include        fastcgi_params;
        fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
    }

    location ~* ^/(css|img|js|flv|swf|download)/(.+)$ {
        root /usr/share/nginx/html/php/blog;
    }

    location ~* \.(eot|ttf|woff|svg|otf)$ {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Headers X-Requested-With;
        add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

### 2.3 在一个域名之下部署多个服务

```conf
server {
    listen       443 ssl;
    server_name api.jkdev.cn;
   
    ssl_certificate   /etc/ssl/jkdev.cn/cert.pem;
    ssl_certificate_key  /etc/ssl/jkdev.cn/key.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    
    location /v1/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    location /v2/ {
        proxy_pass http://127.0.0.1:8081/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    } 
}
```

- location /v1/ ：配置v1项目转发
- location /v2/ ：配置v2项目转发

### 2.4 部署前端项目（如vue）

```conf
server {
    listen       443 ssl;
    server_name www.jkdev.cn;
    root        /usr/share/nginx/html/php/www/dist;
    index       index.php index.html index.htm;
   
    ssl_certificate   /etc/ssl/jkdev.cn/cert.pem;
    ssl_certificate_key  /etc/ssl/jkdev.cn/key.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    location / {                                                                                                                                                              
        try_files $uri $uri/ /index.html;                                                                                                                                     
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }    
 
    location ~* ^/(css|img|js|flv|swf|download)/(.+)$ {
        root /usr/share/nginx/html/php/www/dist;
    }

    location ~* \.(eot|ttf|woff|svg|otf)$ {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Headers X-Requested-With;
        add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
