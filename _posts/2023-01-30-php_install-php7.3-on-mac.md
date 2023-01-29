---
title: 怎样在mac中编译安装php7.3？
date: 2023-01-30 00:28:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2022-04-08，整理时间：2023-01-30。 在撰写本文的时候，php7.3最新的版本是`7.3.33`，以下是macOS编译安装PHP的过程

## 一、编译配置

编译配置，命令执行之后，会检测你系统的环境是否已经满足编译依赖

```shell
./configure \
  --prefix=/usr/local/php-7.3.33 \
  --with-config-file-path=/usr/local/php-7.3.33/etc \
  --with-pdo-mysql=mysqlnd \
  --with-mysqli=mysqlnd \
  --with-libxml-dir \
  --with-gd \
  --with-jpeg-dir \
  --with-png-dir \
  --with-freetype-dir \
  --with-iconv=/usr/local/Cellar/libiconv/1.16 \
  --with-zlib-dir=/usr/local/Cellar/zlib/1.2.11 \
  --with-bz2=/usr/local/Cellar/bzip2/1.0.8 \
  --with-openssl=/usr/local/Cellar/openssl@1.1/1.1.1n \
  --with-curl=/usr/local/Cellar/curl/7.82.0 \
  --enable-soap \
  --enable-mbstring \
  --enable-sockets \
  --enable-exif \
  --with-readline=/usr/local/Cellar/readline/8.1.2 \
  --disable-ipv6
```

以上有些参数指定到了实际的库目录，如`--with-iconv=/usr/local/Cellar/libiconv/1.16`，这里我指定到我电脑里`libiconv`的实际安装目录，如果你电脑没有，可以使用[homebrew](https://brew.sh)进行安装

## 二、编译过程可能遇见的错误

编译的过程中首先可能会遇到以下第两个错误

- liconv相关

```shell
Undefined symbols for architecture x86_64:
  "_libiconv", referenced from:
      _do_convert in gdkanji.o
      _php_iconv_string in iconv.o
      __php_iconv_strlen in iconv.o
      _zif_iconv_substr in iconv.o
      __php_iconv_strpos in iconv.o
      _zif_iconv_mime_encode in iconv.o
      __php_iconv_appendl in iconv.o
      ...
  "_libiconv_close", referenced from:
      _do_convert in gdkanji.o
      _php_iconv_string in iconv.o
      __php_iconv_strlen in iconv.o
      _zif_iconv_substr in iconv.o
      __php_iconv_strpos in iconv.o
      _zif_iconv_mime_encode in iconv.o
      __php_iconv_mime_decode in iconv.o
      ...
  "_libiconv_open", referenced from:
      _do_convert in gdkanji.o
      _php_iconv_string in iconv.o
      __php_iconv_strlen in iconv.o
      _zif_iconv_substr in iconv.o
      __php_iconv_strpos in iconv.o
      _zif_iconv_mime_encode in iconv.o
      __php_iconv_mime_decode in iconv.o
      ...
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [sapi/cli/php] Error 1
```

解决办法：我们手动打开源码目录下 `configure`命令 生成的`Makefile`文件，找到`EXTRA_LIBS`变量，将变量值中的`-liconv`这个参数替换为你实际安装的`libiconv.dylib`的绝对路径，如我的电脑使用`homebrew`安装，在我电脑里绝对路径是`/usr/local/Cellar/libiconv/1.16/lib/libiconv.dylib`。

- readline相关

可能遇到相关的错误如下

```shell
Undefined symbols for architecture x86_64:
  "_append_history", referenced from:
      _readline_shell_run in readline_cli.o
  "_history_list", referenced from:
      _zif_readline_list_history in readline.o
  "_rl_completion_suppress_append", referenced from:
      _zif_readline_info in readline.o
  "_rl_done", referenced from:
      _zif_readline_info in readline.o
  "_rl_mark", referenced from:
      _zif_readline_info in readline.o
  "_rl_pending_input", referenced from:
      _zif_readline_info in readline.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [sapi/cli/php] Error 1
```

找到`EXTRA_LIBS`变量，将变量值中的`-lreadline`这个参数替换为你实际安装的`libreadline.dylib`的绝对路径，如`/usr/local/Cellar/readline/8.1.2/lib/libreadline.dylib`

- 最终修改后

修改了`Makefile`文件之后，最终的`EXTRA_LIBS`变量值如下

```shell
EXTRA_LIBS = -lcrypto -lssl -lcrypto -lresolv /usr/local/Cellar/readline/8.1.2/lib/libreadline.dylib /usr/local/Cellar/libiconv/1.16/lib/libiconv.dylib /usr/local/Cellar/libiconv/1.16/lib/libiconv.dylib -lpng -lz -ljpeg -lbz2 -lz -lcrypto -lssl -lcrypto -lm -lxml2 -lz -licucore -lm -lcurl -lxml2 -lz -licucore -lm -lfreetype -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm
```

## 三、编译与安装

```shell
# 编译
make
# 安装
make install
```
