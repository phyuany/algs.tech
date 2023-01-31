---
title: "Let's Encrypt免费SSL通配证书的申请与续期方法"
date: 2023-01-31 21:33:00 +0800
categories: [工具]
tags: [ssl https]
pin: false
---

## 一、概述

*Let's Encrypt* 是免费、开放和自动化的证书颁发机构。目前有很多网站使用Let's Encrypt证书做https加密。我也一直在用，不过以前都是用的单域名证书，新建网站就需要重新申请，比较麻烦。但现在已经可以申请Let's Encrypt通配证书了。

实际上，申请 Let's Encrypt 生成证书的工具不止一个，我用过 `cerbot` 和 `acme.sh`。以前用 cerbot 申请的时候不得不停掉 80 端口的服务，感觉不太友好。后来用 `acme.sh` 简单一点，本文将介绍如何使用 `acme.sh` 来独立申请域名通配证书。

acme.sh 项目地址为[https://github.com/acmesh-official/acme.sh](https://github.com/acmesh-official/acme.sh)，本文参考官方文档

## 二、安装

### 2.1 安装socat和curl

以下以debian/ubuntu为例，安装命令如下

```shell
apt install socat curl  # debian/ubuntu 环境
```

以下以alpine为例，安装命令如下

```shell
apk add socat curl      # alpine 环境  
```

### 2.2 安装 acmese.sh

```shell
curl  https://get.acme.sh | sh
```

安装完成之后会在当前目录下生成 `.acme.sh` 目录

## 三、生成证书

### 3.1 配置域名服务商的AccessKey

申请域名证书可以手动添加域名解析验证。但使用域名服务商提供的 API 自动添加txt解析的方式来完成验证，会更方便后续实现自动化的ssl证书续订。

根据不同类型的域名服务商，我们选择对应的DNS API ：[https://github.com/acmesh-official/acme.sh/wiki/dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)

首先我们需要获取域名服务商提供的AccessKey，这是用来作为调用API的凭证。因为我的域名是在阿里云买的，这里以阿里云作为示例，可以点击这个链接申请阿里云的Accesskey：[https://usercenter.console.aliyun.com/#/manage/ak](https://usercenter.console.aliyun.com/#/manage/ak)

你可以使用全局的AccessKey，也可以使用子账号的AccessKey。因为全局AccessKey具有的阿里云账号的所有权限，更建议使用子账号。这里我使用子账号的AccessKey（但注意，子账号必须设置好dns相关权限），生成AccessKey ID 和 AccessKey Secret 如下图：

![01.png](/img/tools/12-01.png)

获取AccessKey ID和Access Scret后将它们加入到系统环境变量中

```shell
export Ali_Key="$KEY"
export Ali_Secret="hyddKq5Dm9OBfpCftGRP9Uo3vcFRaa"
```

### 3.2 生成证书

配置好之后，使用以下命令生成证书

```shell
.acme.sh/acme.sh --issue --dns dns_ali -d jkdev.cn -d *.jkdev.cn
```

这里生成两个证书，一个是 jkdev.cn 的，另一个使用 * 来代替，即可生成子域名的通配证书。生成证书过程中，会调用服务商API自动添加一条 txt 的域名解析验证，验证通过后会自动删除解析，这个过程对我们来说是无感知的。

> 注意：如果你没有使用的阿里云的域名，使用上面的命令是不管用的，请参考 3.1 中的 DNS API说明链接。

证书的有效期为90，80天之后续期，续期命令如下

```shell
.acme.sh/acme.sh --renew -d jkdev.cn -d *.jkdev.cn
```

当然了，你可以做成系统任务，自动续期，这里不再说明。

## 四、使用SSL证书部署https网站

### 4.1 部署到Apache服务器

生成的证书可以用于Apache/Nginx等服务器，我用的是Apache，此处共享一下我的配置参数

```xml
<VirtualHost _default_:443>
    DocumentRoot "/var/www/html/www"
    ServerName www.jkdev.cn
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/ssl/test.phy/*.jkdev.cn.cer
    SSLCertificateKeyFile /etc/letsencrypt/live/ssl/test.phy/*.jkdev.cn.key
    SSLCertificateChainFile /etc/letsencrypt/live/ssl/test.phy/fullchain.cer
    <Directory "/var/www/html/www">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
        ErrorDocument 404 https://www.jkdev.cn/404.html
    </Directory>
</VirtualHost>
```

### 4.2 部署到Nginx服务器

以下是Nginx配置文件

```text
server {
    listen       443 ssl;
    server_name admin.jkdev.cn;
    root        /usr/share/nginx/html/php/admin_v2/dist;
    index       index.php index.html index.htm;
   
    ssl_certificate   /etc/ssl/jkdev.cn/cert.pem;
    ssl_certificate_key  /etc/ssl/jkdev.cn/key.pem;
    ssl_session_timeout 5m; #缓存有效期
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #安全链接可选的加密协议
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; #加密算法
    ssl_prefer_server_ciphers on; #使用服务器端的首选算法

    location / {                                                                                                                                                              
        try_files $uri $uri/ /index.html;                                                                                                                                     
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }    
 
    location ~* ^/(css|img|js|flv|swf|download)/(.+)$ {
        root /usr/share/nginx/html/php/admin_v2/dist;
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

有任何疑问，欢迎给留言评论！
