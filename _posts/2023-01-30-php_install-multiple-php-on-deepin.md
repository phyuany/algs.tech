---
title: 在deepin上安装多个PHP版本
date: 2023-01-30 00:28:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2022-12-20，整理时间：2023-01-30

在 deepin 20 上使用编译的方式安装多个版本

## 一、安装依赖

首先安装依赖

```shell
sudo apt install -y libcurl4-openssl-dev libgd-dev libwebp-dev libpng++-dev libfreetype6-dev libghc-zlib-dev libmcrypt-dev libxml++2.6-dev libssl-dev libbz2-dev libedit-dev libreadline-dev libsqlite3-dev libonig-dev libzip-dev
```

因为PHP7需要的freetype版本要低一些，手动编译安装如下

```shell
wget https://download.savannah.gnu.org/releases/freetype/freetype-2.8.1.tar.gz
tar zxvf freetype-2.8.1.tar.gz
cd freetype-2.8.1/
./configure --prefix=/usr/local/freetype-2.8.1/
```

需要注意的是，以下示例中，需要用到的php源码直接从php官网下载。

## 二、安装PHP

### 2.1 PHP 7.2

```shell
# 编译检测
./configure \
  --prefix=/usr/local/php-7.2.34 \
  --with-config-file-path=/usr/local/php-7.2.34/etc \
  --with-pdo-mysql=mysqlnd \
  --with-mysqli=mysqlnd \
  --with-libxml-dir \
  --with-gd \
  --with-jpeg-dir \
  --with-png-dir \
  --with-freetype-dir=/usr/local/freetype-2.8.1 \
  --with-iconv-dir \
  --with-zlib-dir \
  --with-bz2 \
  --with-openssl \
  --with-curl \
  --enable-pcntl \
  --with-readline \
  --enable-soap \
  --enable-zip \
  --enable-mbstring \
  --enable-sockets \
  --enable-exif
# 编译
make -j16
# 安装
make install
```

### 2.2  PHP 7.3

```shell
# 编译检测
./configure \
  --prefix=/usr/local/php-7.3.33 \
  --with-config-file-path=/usr/local/php-7.3.33/etc \
  --with-pdo-mysql=mysqlnd \
  --with-mysqli=mysqlnd \
  --with-libxml-dir \
  --with-gd \
  --with-jpeg-dir \
  --with-png-dir \
  --with-freetype-dir=/usr/local/freetype-2.8.1 \
  --with-iconv \
  --with-zlib-dir \
  --with-bz2 \
  --with-openssl \
  --with-curl \
  --enable-soap \
  --enable-mbstring \
  --enable-sockets \
  --enable-exif \
  --enable-pcntl \
  --enable-zip \
  --with-readline
# 编译
make -j16
# 安装
make install
```

### 2.3 PHP 8.0

```shell
# 编译
./configure --prefix=/usr/local/php-8.0.25 \
  --with-config-file-path=/usr/local/php-8.0.25/etc \
  --with-zlib \
  --with-zip \
  --with-pdo-mysql=mysqlnd \
  --with-mysqli=mysqlnd \
  --with-mysqli=mysqlnd \
  --enable-gd \
  --with-external-gd \
  --with-jpeg \
  --with-xpm \
  --with-webp \
  --with-freetype \
  --with-zlib-dir \
  --with-bz2 \
  --with-openssl \
  --with-curl \
  --enable-soap \
  --enable-pcntl \
  --enable-mbstring \
  --enable-sockets \
  --enable-exif \
  --enable-pcntl \
  --with-readline
# 编译
make -j16
# 安装
sudo make install
```

### 2.4 PHP 5.6

`php 5.6` 这个版本相对老，因此有一些依赖库依赖的是旧版本。我在 `Deepin 20.9`上安装时，`curl` 和 `openssl` 这两个依赖库需要安装旧版本，我编译安装了 `curl-7.19.7` 和 `openssl-1.0.2u`。

- 安装 curl

源码下载页面：<https://www.openssl.org/source/old/>

编译与安装命令如下

```shell
# 编译配置
./configure --prefix=/usr/local/curl-7.19.7
# 编译
make -j
# 安装
sudo make install
```

- 安装 opelssl

源码下载页面：<https://www.openssl.org/source/old/>

编译与安装命令如下

```shell
# 编译配置
./config --prefix=/usr/local/openssl-1.0.2u
# 编译
make -j
# 安装
sudo make install
```

- 安装 php

```shell
# 编译
./configure \
    --prefix=/usr/local/php-5.6.40 \
    --with-config-file-path=/usr/local/php-5.6.40/etc \
    --with-mysql \
    --with-mysqli \
    --with-pdo-mysql \
    --with-zlib \
    --with-curl=/usr/local/curl-7.19.7 \
    --with-mcrypt \
    --with-gd \
    --with-jpeg-dir \
    --with-freetype-dir=/usr/local/freetype-2.8.1 \
    --with-openssl=/usr/local/openssl-1.0.2u \
    --with-mhash \
    --disable-rpath \
    --disable-ipv6 \
    --enable-fpm \
    --enable-pcntl \
    --enable-gd-native-ttf \
    --enable-sockets \
    --enable-bcmath
# 编译
make -j16
# 安装
sudo make install
```

### 2.3 PHP 8.3

```shell
# 编译配置
./configure --prefix=/usr/local/php-8.3.0 \
  --with-config-file-path=/usr/local/php-8.3.0/etc \
  --with-zlib \
  --with-zip \
  --with-pdo-mysql=mysqlnd \
  --with-mysqli=mysqlnd \
  --with-mysqli=mysqlnd \
  --enable-gd \
  --with-external-gd \
  --with-jpeg \
  --with-xpm \
  --with-webp \
  --with-freetype \
  --with-zlib-dir \
  --with-bz2 \
  --with-openssl \
  --with-curl \
  --enable-soap \
  --enable-pcntl \
  --enable-mbstring \
  --enable-sockets \
  --enable-exif \
  --enable-pcntl \
  --enable-fpm \
  --with-readline
# 编译
make -j4
# 安装
sudo make install
```
