---
title: 如何使用Apache做反向代理
date: 2023-01-31 21:19:00 +0800
categories: [工具]
tags: []
pin: false
---

> 本文撰写时间：2018-07-21 09:11，修改时间：2022-07-12

## 一、反向代理

- 正向代理：服务器A 替代客户端去请求另外一台服务器B，将数据返回客户端，这个过程叫正向代理，正向代理隐藏了真实的客户端

- 反向代理：服务器A 将客户端的请求转发给了服务器B去处理，客户端不知道服务器B的存在，这个过程叫反向代理，反向代理隐藏了真实的服务器

反向代理简单理解如下图示：

![timg.jpeg](/img/tools/05-01.png)

在同一台服务器内，假设服务器对外网只开放了80端口，当客户端请求80端口的http服务，转发给其他端口服务来处理，这个过程也是反向代理。这篇博客给大家演示Apache反向代理设置，使的得一台服务器上同时支持运行多端口服务（如同时运行PHP和Java项目）。

## 二、工具

- 1. Apache 2.4.33

- 2. JDK 1.8 + Tomcat 8.5

## 三、步骤

### 3.1、开启mod_proxy.so和mod_proxy_http.so模块

在Apache的`httpd.conf`配置文件中，添加以下两行配置

```shell
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

### 3.2、设置转发

进入Apache安装目录下的/conf/extra目录，如果你使用的http默认端口，更改httpd-vhosts.conf文件，

```xml
<VirtualHost _default_:80>
    ServerName test.cn #域名
    
    ProxyPreserveHost On # 开启反向代理
    ProxyRequests Off # 关闭正向代理
    
    ProxyPass / http://127.0.0.1:8080/ #映射地址
    ProxyPassReverse / http://127.0.0.1:8080/ #映射302重定向的地址
</VirtualHost>
```

假如使用的是https加密端口的话，更改httpd-ssl.conf文件，如下

```xml
<VirtualHost _default_:443>
    ServerName test.cn #域名
    SSLEngine on
    SSLCertificateFile "证书所在目录/fullchain.pem"
    SSLCertificateKeyFile "证书所在目录/privkey.pem"
    
    ProxyPreserveHost On
    ProxyRequests Off
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost> 
```

保存配置重启Apache，在浏览器输入直接输入域名就可以访问到8080端口的项目了
