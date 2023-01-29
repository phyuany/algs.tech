---
title: Linux下安装Redis
date: 2023-01-30 00:06:00 +0800
categories: [NoSQL]
tags: []
pin: false
---

> 撰写时间：2018-01-15，整理时间：2023-01-30

## 一、下载与安装

下文是在Linux安装redis的过程，使用root用户进行操作

```shell
#下载源代码
wget http://download.redis.io/releases/redis-2.8.17.tar.gz
#解压
tar xzf redis-2.8.17.tar.gz
#移动文件夹到/usr/local目录
mv redis-2.8.17 /usr/local/redis
#编译
make
```

make完后 redis目录下会出现编译后的redis服务程序redis-server，还有用于测试的客户端程序redis-cli，两个程序位于安装目录 src 目录下

## 二、测试数据库

进入redis目录，执行以下命令：

```shell
#参数指定配置文件
./src/redis-server redis.conf
```

启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了。 如：

```shell
./src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

## 三、关闭数据库服务

关闭命令为：`redis-cli -p 端口号 shutdown`，如下：

```shell
./src/redis-cli -p 6379 shutdown
```

## 四、修改配置文件redis.conf

- （1）设置支持远程访问：

找到以下一行

```shell
bind 127.0.0.1
```

注释当前行即可支持远程登录

```shell
# bind 127.0.0.1
```

- （2）设置密码：

redis默认没有密码，使用下列方式设置密码，找到下面一行

```shell
# requirepass foobared
```

取消注释，“foobared”就是密码，可以改成其他的，如改成“myredis”

```shell
requirepass myredis
```

重启服务，使用a参数传入密码才可以正常操作

```shell
./src/redis-cli -a myredis
```
