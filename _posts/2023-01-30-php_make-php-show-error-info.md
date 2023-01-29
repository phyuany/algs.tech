---
title: 怎样让PHP提示错误信息
date: 2023-01-30 00:10:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2019-03-04，整理时间：2023-01-30

## 一、概述

从源代码安装PHP的默认情况下，在运行代码时如果出现错误不提示任何错误信息，其实我们只有修改以下PHP的配置文件就可以让它显示错误信息了

## 二、环境

- 1. Ubuntu 16.04 操作系统
- 2. 从源码安装的 PHP 7.2.7

## 三、解决步骤

首先使用编辑工具打开 php 配置文件，我的php安装目录是`/usr/local/php`，配置文件的目录在该目录下的`/etc`目录

```shell
vim /usr/local/php/etc/php.ini
```

在php配置文件的头部添加以下几行代码

```shell
# ini_set函数作用：为一个配置选项设置值，
ini_set("display_errors", "stderr");
# error_reporting是设置错误报告内容，E_ALL代表显示所有错误
error_reporting(E_ALL);
```

如下图所示：

![深度截图_20190304083611.png](/img/php/03-01.png)

参数说明：

- display_errors

在php的配置文件中，"display_errors"选项设置作用是：是否将错误信息作为输出的一部分显示到屏幕，或者对用户隐藏而不显示。设置 "stderr" 表示发送到 stderr 而不是 stdout。 "stderr"从 PHP 5.2.4 开始可用。在以前的版本中，该配置值的类型为 boolean。

`stdout`（标准输出），输出方式是行缓冲。输出的字符会先存放在缓冲区，等按下回车键时才进行实际的I/O操作。

`stderr`（标准出错），是不带缓冲的，这使得出错信息可以直接尽快地显示出来。

尽管 display_errors 也可以在运行时设置 (使用 ini_set())， 但是脚本出现致命错误时任何运行时的设置都是无效的。 因为在这种情况下预期运行的操作不会被执行。

- error_reporting

设置错误报告的级别。该参数可以是一个任意的表示二进制位字段的整数，或者常数名称。错误级别和常数是在 预定义常量定义的。

在 PHP5.3 及以上版本中，默认值为 `E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED`。 该设置不会显示 `E_NOTICE`、 `E_STRICT` 、`E_DEPRECATED` 级错误提示。

在 PHP 5.3.0 以前版本中，默认值是 `E_ALL & ~E_NOTICE & ~E_STRICT`。

在 PHP 4 中，默认值是 `E_ALL & ~E_NOTICE`。

### 四、重启服务器

重启Apache服务器或者PHP-FPM之后，即可看到运行错误代码时会提示错误信息

> 参考php官方文档：[http://php.net/manual/zh/errorfunc.configuration.php#ini.display-errors](http://php.net/manual/zh/errorfunc.configuration.php#ini.display-errors)
