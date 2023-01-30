---
title: LAMP环境搭建之从源码安装PHP
date: 2023-01-31 00:10:00 +0800
categories: [服务器]
tags: []
pin: false
---

## 一、环境

1. Ubuntu 16.04 Server
2. php-7.3.4.tar.gz 源码包

> PHP官网源码下载链接：[https://www.php.net/downloads.php](https://www.php.net/downloads.php)

在Apache中安装PHP，就是让Apache支持对PHP代码的解析，废话不多说，现在开始。

## 二、安装步骤

1. 安装php依赖环境
2. 配置编译参数
3. 编译与安装
4. 配置Apache支持PHP解析
5. 使用测试

## 三、安装过程

所有在root用户模式下进行，“#”后为注释内容

### 3.1. 安装php依赖环境

解压源码

```shell
# 解压源代码
tar -zxvf php-7.3.4.tar.gz
# 进入源码目录
cd php-7.3.4
```

解压之后通过ls命令可以看到源码目录下有很多文件，如下图
![2019042401.png](/img/server/02-01.png)

目录下有一个configure可执行文件，用于配置编译参数

本次安装，我们要实现注意到达以下目标

- 加载所有模块
- 设置MySQL驱动为mysqlnd
- 添加gd、webp、jpeg、png图片库的支持，可以对图片进行处理、生成图片验证码等
- 添加curl的支持，用于进行网络请求
- 添加freetype字体库的支持
- 添加zlib的支持，用于进行数据压缩
- 添加soap的支持，SOAP是一种简单的基于XML的协议，它使应用程序通过HTTP来交换信息。添加soap扩展用来编写soap服务器和客户端
- 添加mbstring的支持，多字节字符串
- 添加sockets的支持
- 添加exif的支持
- 取消ipv6的支持，目前ipv6目前没有完全普及，使用ipv6有时候会出现一些问题，直接禁用
- 添加libmcrypt的支持
- 添加xml支持
- 添加openssl的支持
- 添加对bzip2的支持

实际上，根据你的项目需求，你可能需要配置更多参数，以上只是一个最常规的演示，你可以使用`./configure -help`命令查看详细说明。不过，对于学习PHP基础搭建的环境，上面的配置原则应该够用了。

根据上面需求，我们需要在系统中安装一些依赖库，如下命令：

```shell
apt install libcurl4-openssl-dev #安装curl开发套件
apt install libgd-dev #安装gd开发套件
apt install libwebp-dev #安装webp开发套件
apt install libjpeg-dev #安装jpeg开发套件
apt install libpng++-dev #安装png开发套件
apt install libfreetype6-dev #安装libfreetype6-dev开发套件
apt install libghc-zlib-dev #安装zlib开发套件
apt install libmcrypt-dev #安装libmcrypt开发套件
apt install libxml++2.6-dev #安装libxml开发套件
apt install libssl-dev #安装ssl开发套件
apt install libbz2-dev #安装bzip2开发套件
```

或者执行以下一次性命令

```shell
apt install -y libcurl4-openssl-dev libgd-dev libwebp-dev libpng++-dev libfreetype6-dev libghc-zlib-dev libmcrypt-dev libxml++2.6-dev libssl-dev libbz2-dev
```

### 3.2 配置编译参数

根据3.1中的目标说明，我们执行以下参数配置命令

```shell
./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/php/etc --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-libxml-dir --with-gd --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-zlib-dir --with-bz2 --with-openssl --with-curl --enable-soap --enable-mbstring --enable-sockets --enable-exif --disable-ipv6
```

配置过程如果没有错误的话，可以看到以下如图所示：
![2019042402.png](/img/server/02-02.png)

关键参数说明：

**--prefix**：指定安装位置
**--with-apxs2**：指定Apache 2中apxs模块所在的目录，我们在上一节安装了Apache 2.4.39
**--with-config-file-path**：指定php配置文件所在目录
**其他**：其他的就不细说了，这些参数就对应在二、1中说的那些

### 3.3 添加虚拟内存

编译过程会消耗比较多的内存，如果服务器内存比较小（小于1G的情况），创建交换空间，可以再创建1G的swap空间（从硬盘分配的虚拟内存空间），如下命令

```shell
dd if=/dev/zero of=/mnt/swap bs=1M count=1024 
```

设置交换分区文件

```shell
mkswap /mnt/swap
```

启动swap

```shell
swapon /mnt/swap
```

设置开机时自启用 swap 分区，需要修改文件 /etc/fstab中，添加以下一行配置

```shell
/mnt/swap swap swap defaults 0 0
```

### 3.4 编译与安装

参数配置结束之后源码目录会多出一个Makefile文件，我们就可以编译安装了，执行以下命令

```shell
make #编译
make install #安装
```

在2中我们在参数配置中指定配置文件的目录为：`/usr/local/php/etc`，因此把源码目录下的实例配置文件复制到我们指定的配置文件目录中，执行以下命令

```shell
cp php.ini-production /usr/local/php/etc/php.ini
```

### 3.5 设置Apache支持PHP解析

使用vim编辑Apache服务器的配置文件

```shell
vim /usr/local/apache2/conf/httpd.conf
```

- 添加 php 得支持

在`<IfModule mime_module>`添加以下内容

```shell
AddType application/x-httpd-php .php
```

另外，找到如下内容

```xml
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

改为

```xml
<IfModule dir_module>
    DirectoryIndex index.html index.htm index.php
</IfModule>
```

- 修改服务器名称

找到如下内容

```ini
#ServerName www.example.com:80
```

去掉注释，修改为

```ini
ServerName localhost:80
```

### 3.6 使用测试

我们重启Apache服务器之后。在Apache的htdocs目录下，添加`index.php`文件用于测试PHP是否正常解析，在文件中加入以下内容

```php
<?php
phpinfo();
```

保存之后，打开浏览器，打开如下链接，以下"$IP"替换成你的实际IP或者域名

`http://$IP/index.php`

如果看到如下图中结果，代表安装成功了

![2019042403.png](/img/server/02-03.png)
