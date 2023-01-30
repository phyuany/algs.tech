---
title: Apache服务器VirtualHost常用配置
date: 2023-01-31 00:15:00 +0800
categories: [服务器]
tags: []
pin: false
---

Apache服务器中的VirtualHost用来定义虚拟网站，我们可以在一个VirtualHost中定义一个项目（网站），也可以使用反向代理的方式定义多个项目（即一个域名之下多个子项目）。以下总结Apache服务器VirtualHost常用配置。

## 一、常规配置

```xml
<VirtualHost *:80>
        DocumentRoot "/var/www/html"
        ServerName www.jkdev.cn
</VirtualHost>
```

- DocumentRoot：网站目录
- ServerName：网站域名

## 二、常用附加配置

```xml
<VirtualHost *:80>
       DocumentRoot "/var/www/html"
       ServerName localhost
       <Directory "/var/www/html">
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
           ErrorDocument 404 https://www.jkdev.cn/404.html
       </Directory>
</VirtualHost>
```

- Directory节点：定义目录属性
- Options Indexes FollowSymLinks：开启目录访问，显示目录结构，并允许在此目录中使用符号连接
- AllowOverride All：允许定义.htaccess文件
- AllowOverride None：忽略.htaccess文件
- Require all granted：允许所有请求
- ErrorDocument 404 https://www.jkdev.cn/404.html：路径匹配时跳转的404界面

## 三、开启HTTPS

```xml
<VirtualHost _default_:443>
    DocumentRoot "/var/www/html"
    ServerName hook.jkdev.cn
    SSLEngine on
    SSLCertificateFile /etc/ssl/2_hook.jkdev.cn.crt
    SSLCertificateKeyFile /etc/ssl/3_hook.jkdev.cn.key
    SSLCertificateChainFile /etc/ssl/1_root_bundle.crt
    <Directory "/var/www/html">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
        ErrorDocument 404 https://www.jkdev.cn/404.html
    </Directory>
    </VirtualHost>
```

- SSLEngine on：开启SSL
- SSLCertificateFile：指定证书公钥文件
- SSLCertificateKeyFile：指定证书私钥文件
- SSLCertificateChainFile：指定证书链文件

## 四、反向代理（端口转发）

```xml
<VirtualHost _default_:443>
    ServerName api.jkdev.cn
    SSLEngine on
    SSLCertificateFile /etc/ssl/api/2_api.jkdev.cn.crt
    SSLCertificateKeyFile /etc/ssl/3_api.jkdev.cn.key
    SSLCertificateChainFile /etc/ssl/1_root_bundle.crt

    ProxyPreserveHost On
    ProxyRequests Off

    ProxyPass /v2 http://localhost:92/
    ProxyPass /v1 http://localhost:91/
    ProxyPass / http://localhost:90/
        
    ProxyPassReverse /v2 http://localhost:92/
    ProxyPassReverse /v1 http://localhost:91/
    ProxyPassReverse / http://localhost:90/
</VirtualHost>
```

- ProxyPreserveHost On：开启反向代理
- ProxyRequests Off：关闭正向代理
- ProxyPass：设置反向代理路径
- ProxyPassReverse：使Apache自动处理反向代理中的从定向响应，一般和ProxyPass一起用

## 五、http重定向到https

```xml
<VirtualHost *:80>
    ServerName jkdev.cn
    #redirect
    RewriteEngine on
    RewriteCond %{SERVER_PORT} !^443$
    RewriteRule ^(.*)?$ https://%{SERVER_NAME}$1 [L,R]
</VirtualHost>
```

- RewriteEngine on：开启从定向功能
- RewriteCond %{SERVER_PORT} !^443$：从定向条件，端口不是443时重定向，^为开头，$为结束
- RewriteRule：重定向规则，L：表明当前规则是最后一条规则，停止分析以后规则的重写；R：强制外部重定向
