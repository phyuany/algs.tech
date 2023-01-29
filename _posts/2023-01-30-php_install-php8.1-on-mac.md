---
title: 怎样在mac中编译安装php8.1？
date: 2023-01-30 00:28:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2022-04-08，整理时间：2023-01-30。 在撰写本文的时候，php的最新稳定版本是`8.1.4`，本文介绍macOS下编译安装方法。

## 一、配置编译参数

```shell
./configure --prefix=/usr/local/php-8.1.4 \
--with-config-file-path=/usr/local/php-8.1.4/etc \
--with-zlib \
--with-zip \
--with-pdo-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-bz2=/usr/local/Cellar/bzip2/1.0.8 \
--with-iconv=/usr/local/Cellar/libiconv/1.16 \
--enable-gd \
--with-external-gd \
--with-jpeg \
--with-xpm \
--with-webp \
--with-freetype \
--with-zlib-dir \
--with-openssl \
--with-curl \
--enable-soap \
--enable-mbstring \
--enable-sockets \
--with-readline=/usr/local/Cellar/readline/8.1.2 \
--enable-exif
```

以上有些参数指定到了实际的库目录，如`--with-iconv=/usr/local/Cellar/libiconv/1.16`，这里我指定到我电脑里`libiconv`的实际安装目录，如果你电脑没有，可以使用[homebrew](https://brew.sh)进行安装

## 二、编译与安装

在编译之前先加上openssl库里`pkgconfig`命令所在目录添加到`PKG_CONFIG_PATH`环境变量，否则可能报错，如下命令

```shell
export PKG_CONFIG_PATH=/usr/local/Cellar/openssl@1.1/1.1.1n/lib/pkgconfig:$PKG_CONFIG_PATH
```

编译与安装

```shell
# 编译
make
# 安装
make install
```
