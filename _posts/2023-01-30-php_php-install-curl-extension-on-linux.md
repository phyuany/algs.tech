---
title: Linux下php安装curl扩展
date: 2023-01-30 00:09:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2018-03-22，整理时间：2023-01-30

## 一、使用php -m查看php当前已开启扩展库

php使用源码编译安装，原本在安装php时没有设置curl扩展库，最近项目需要curl扩展库的支持，于是查看php是否已经开启curl扩展库

```shell
/usr/local/php/bin/php -m
```

结果如图所示，没有curl扩展：
![01.png](/img/php/02-01.png)

## 二、安装curl扩展库

进入源码目录，进入到php源码目录下的扩展库源码目录的curl源码目录，源码目录是当时安装php时使用的源码的目录，如果删除了，可以去php官方重新下载解压，可以看到以下文件，如下图：

![02.png](/img/php/02-02.png)

调用phpize程序生成编译配置文件：这个命令可能会产生一些错误，主要是因为你的系统里缺少一些库，你需要按照提示安装缺失的库就可以了,调用phpize程序后可以看到多出了几个文件，最关键的是`configure`文件，如图所示：

![03.png](/img/php/02-03.png)

调用`configure`生成Makefile文件，--with-php-config的值是：php安装路径中的`/bin/php-config`，在执行configure的过程中，会检测系统的依赖是否满足，如果依赖不满足将会报错，按提示解决即可：

```shell
./configure -with-curl=/usr/local/curl --with-php-config=/usr/local/php/bin/php-config
```

执行完configure，出现Makefile文件后先调用make编译，`make install`安装，如果没有生成Makefile文件，就是系统缺失库，根据提示安装后在此执行configure命令即可:

```shell
make && make install
```

如图所示就是安装成功了，即生成了curl.so文件，位置就在`/usr/local/php/lib/php/extensions/no-debug-zts-20160303/`目录下，如下图：

![04.png](/img/php/02-04.png)

我们需要添加php对curl的支持，编辑php.ini文件在文件末尾添加一下加载代码，extense的值是curl.so的路径，然后保存退出即可：

![05.png](/img/php/02-05.png)

重启`Apache服务器`或者`php-fpm`即可生效，如果再次调用`php -m`命令，可以看到已经多出了curl扩展了：

![06.png](/img/php/02-06.png)
