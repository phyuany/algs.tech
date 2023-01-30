---
title: LAMP环境搭建之从源码安装Apache
date: 2023-01-31 00:09:00 +0800
categories: [服务器]
tags: []
pin: false
---

> 撰写时间：2019.04.21

## 一、准备

1. Ubuntu 16.04 Server 纯净系统
2. Apache httpd-2.4.39 源码

Apache的httpd软件，通常我们直接叫做Apache服务器。

Apache httpd-2.4.39的下载地址：[http://httpd.apache.org/download.cgi](http://httpd.apache.org/download.cgi)，我在撰写此文章的时候这是最新版本。你可以下载.tar.bz2压缩包，也可以.tar.gz压缩包。文章中使用的是`httpd-2.4.39.tar.gz`

## 二、概述

### 2.1 从软件库快速安装

Apache是一个大名鼎鼎的服务器软件，我们实际上可以直接从Linux软件库中安装，如下命令

在Fedora/CentOS/Red Hat上安装Apache与运行Apache

```shell
# 安装
sudo yum install httpd
# 设置开机自启
sudo systemctl enable httpd
# 启动服务
sudo systemctl start httpd
```

在Ubuntu/Debian上安装并运行Apache

```shell
# 安装服务
sudo apt install apache2
# 启动服务
sudo service apache2 start
```

### 2.2 为什么需要源码安装？

不过有小伙伴可能有疑问，两条命令不是直接安装成功了吗？为什么还要使用从源代码安装的方式来安装Apache？

答：观看教程的小伙伴大多是初学者，使用从源码安装的方式，让他们进一步了解底层的一些知识。如果你喜欢从软件库安装，初学者可能完全不知道安装过程。对自己后面使用和理解Apache都是一组阻碍。所以建议的是：初学者最好使用源码安装。

### 2.3 安装过程

- 安装程序编译环境

C/C++源代码需要通过编译生成可执行文件，才可以正常运行。而Apache服务器软件就是使用C/C++开发的，所以我们先需要先编译，所以编译环境必须有，你懂的。

- 配置编译参数

Apache源码目录下的configure可执行文件，就是用来配置安装Apache的编译参数，生成编译配置文件`Makefile`。通过配置参数，我们可以自定义一些Apache服务器的一些功能，在配置参数的过程中，Apache会检查你系统缺失的必备程序组件，如果Linux系统缺失相应组件，则会输出相关错误提示，需要手动解决。解决过程无非就是把缺失的内容安装上。

- 编译与安装

执行`configure`可执行文件生成make编译配置文件`Makefile`之后，就可以编译、安装了。废话少说，开始吧。

## 三、安装基础环境

对于安装Apache需要依赖的环境，在Fedora/CentOS/Red Hat上使用yum工具来安装依赖程序包，在Ubuntu/Debian上使用apt工具来安装依赖程序包。当然了，你的Linux系统需要50M以上的临时空间、你的Linux系统能连接网络，同时你的系统时间正确。

根据官方文档，从httpd-2.4.39源码安装，确保你系统包含以下环境。本文以Ubuntu为例，下面是按照基础环境的过程

- APR and APR-Util

确保您的系统上已安装APR和APR-Util，在某些系统上需要安装相应的dev包。

```shell
sudo apt install -y libaprutil1-dev
```

- PCRE

```shell
sudo apt install -y libpcre3-dev
```

- C编译器和构建系统
这里我们需要把gcc和cmake都安装上

```shell
sudo apt install -y gcc cmake
```

- Perl 5

```shell
sudo apt install -y perl
```

- openssl

这个程序不是官方不强制安装的。在此教程的后续，我会写一个安装ssl证书的教程，所以我们再此安装Apache服务器之前我们先安装上openssl。

```shell
sudo apt install -y openssl
```

- 安装所有环境

如果觉得麻烦，可以一次性安装上面提到的所有依赖，如下命令：

```shell
sudo apt install -y libaprutil1-dev libpcre3-dev gcc cmake perl openssl
```

## 四、安装过程

以下操作都在root用户模式下进行，“#”后为注释内容。

- 配置编译参数

“#”号后面为注释内容

```shell
# 解压源码包
tar -zxvf httpd-2.4.39.tar.gz
# 进入源码目录
cd httpd-2.4.39
# 查看源码目录内容
ls
```

你可以看到以下文件，如图所示：
![1980420.png](/img/server/01-01.png)

- 配置编译参数

```shell
./configure --prefix=/usr/local/apache2  --enable-modules=all --enable-mods-shared=all --enable-so
```

confirgure参数说明：

（1）**--prefix**：指定安装目录，如果不指定的话，默认就是`/usr/local/apache2`；

（2）**--enable-modules=all**：加载所有模块；

（3）**--enable-mods-shared=all**：所有模块使用动态编译的方式进行编译。如果不加此参数，默认为静态编译，静态是直接编译进httpd中， 动态编译会提供一个module.so 文件，需要在httpd.conf配置文件中使用时用 loadmodule 这个语法来加载；

（4）**--enable-so**：其实使用动态编译方式时该模块会自动生效。

本次安装我们使用以上列出的参数配置，就可以在很多需求场景使用了，不过你实际的项目上可能会做修改。具体参数说明你可以使用`./configure -help`查看。

参数配置结束之后，会看到如下图信息：
![19042101.png](/img/server/01-02.png)

此时，源码目录下面多出一个Makefile文件，但是，如果你的环境和我演示的不一样，执行上面过程你可能不会顺利通过，那么就需要你解决相应的错误了。有问题，如果遇到相关问题不能自己网上上搜解决，也可以私信到我。

- 编译与安装

```shell
# 编译
make
# 安装
make install
```

如果没有任何错误，那就是安装成功了，此时，`/usr/local/`目录下多出了apache2目录，如图所示：
![19042103.png](/img/server/01-03.png)

也就是我们安装apache的目录。此时，执行以下命令对apache服务器进行相应操作

```shell
# 开启服务器
/usr/local/apache2/bin/apachectl start
# 关闭服务器
/usr/local/apache2/bin/apachectl stop
# 重启服务器
/usr/local/apache2/bin/apachectl restart
```

开启服务之后，打开网络浏览器，输入服务器的公网IP地址，看到浏览器“It Works”字样，那么恭喜你，安装成功了。

此次文章到这里，本系列教程共三篇，本文是第一篇，欢迎持续关注。
