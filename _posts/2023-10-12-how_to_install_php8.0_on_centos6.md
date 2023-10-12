---
title: 怎样在centos6上安装PHP 8.0
date: 2023-10-12 13:10:00 +0800
categories: [PHP]
tags: [php8.0]
pin: false
---

## 更新yum

```shell
# yum
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos6_base.repo
# epel（RHEL6系列）
wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-6.repo

# 更新缓存
yum clean all
yum makecache
```

## libicov

下载网址：<libiconv:https://ftp.gnu.org/gnu/libiconv/>

编译与安装

```shell
./configure --prefix=/usr/local/libiconv-1.16
make && make install
```

## libxml2

下载网址：<https://download.gnome.org/sources/libxml2/2.9/>

编译与安装

```shell
./configure --prefix=/usr/local/libxml2-2.9.14 --with-history --with-python=/usr/bin --docdir=/usr/local/libxml2-2.9.14/doc
make && make install
```

## openssl

下载网址：<https://www.openssl.org/source/openssl-1.1.1w.tar.gz>

编译与安装

```shell
./Configure no-shared linux-x86_64 no-md2 no-mdc2 no-rc5 no-rc4 enable-ssl2 enable-ssl3 --prefix=/usr/local/openssl-1.1.1w
make && make install
```

## sqlite

下载网址：<https://github.com/sqlite/tags?after=version-3.40.0>

```shell
./configure --prefix=/usr/local/sqlite3-3.40.1
make && make install
```

## curl

下载网址：<https://curl.se/download/curl-7.83.0.tar.gz>

```shell
./configure --prefix=/usr/local/curl-7.83.0 --with-ssl=/usr/local/openssl-1.1.1w
make && make install
```

## oniguruma

下载网址：<https://github.com/kkos/oniguruma/releases/tag/v6.9.8>

编译与安装

```shell
./configure --prefix=/usr/local/onig-6.9.8
make && make install
```

## libpng16

下载网址：<https://udomain.dl.sourceforge.net/project/libpng/libpng16/1.6.37/libpng-1.6.37.tar.gz>

编译与安装

```shell
./configure --prefix=/usr/local/libpng-1.6.37
make && make install
```

## gd

下载网址：<https://github.com/libgd/libgd/releases>

编译与安装

```shell
./configure --prefix=/usr/local/libgd-2.3.3 --with-png=/usr/local/libpng-1.6.37
make && make install
```

## cmake

下载网址：<https://cmake.org/files/v3.23/cmake-3.23.2-linux-x86_64.tar.gz>

安装

```shell
mv cmake-3.23.2-linux-x86_64 /usr/local/cmake-3.23.2
```

## libzip

下载网址：<https://libzip.org/download/libzip-1.10.1.tar.gz>

编译与安装：

```shell
/usr/local/cmake-3.23.2/bin/cmake -DCMAKE_INSTALL_PREFIX=/usr/local/libzip-1.8.0
make
make install
```

## 安装php

编译与安装：

```shell
./configure --prefix=/usr/local/php-8.0.30 --with-config-file-path=/usr/local/php-8.0.30/etc --with-zlib-dir --enable-mbstring --with-curl --with-zlib --with-freetype=/usr/local/freetype-2.9/ --disable-rpath --enable-sockets --enable-sysvsem --enable-sysvshm --enable-pcntl --enable-mbregex --with-mhash --with-pdo-mysql --with-mysqli --with-openssl --enable-fpm --with-pdo-mysql --enable-ftp --enable-gd --without-iconv --disable-opcache LIBXML_CFLAGS=-I/usr/local/libxml2-2.9.14/include/libxml2 LIBXML_LIBS="-L/usr/local/libxml2-2.9.14/lib -lxml2" OPENSSL_CFLAGS=-I/usr/local/openssl-1.1.1w/include OPENSSL_LIBS="-L/usr/local/openssl-1.1.1w/lib -lcrypto -lssl" SQLITE_CFLAGS=-I/usr/local/sqlite3-3.40.1/include SQLITE_LIBS="-L/usr/local/sqlite3-3.40.1/lib -lsqlite3" CURL_CFLAGS=-I/usr/local/curl-7.83.0/include CURL_LIBS="-L/usr/local/curl-7.83.0/lib -lcurl" ONIG_CFLAGS=-I/usr/local/onig-6.9.8/include ONIG_LIBS="-L/usr/local/onig-6.9.8/lib -lonig" GD_CFLAGS=-I/usr/local/libgd-2.3.3/include GD_LIBS=-L/usr/local/libgd-2.3.3/lib PNG_CFLAGS=-I/usr/local/libpng-1.6.37/include PNG_LIBS="-L/usr/local/libpng-1.6.37/lib -lpng" LIBZIP_CFLAGS=-I/usr/local/libzip-1.8.0/include LIBZIP_LIBS=-L/usr/local/libzip-1.8.0/lib64
```
