---
title: Linux怎样安装MongoDB
date: 2023-01-30 00:05:00 +0800
categories: [NoSQL]
tags: []
pin: false
---

> 撰写时间：2018-01-15，整理时间：2023-01-30

## 1、概述

MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档结构为 JSON ，官方叫做 BJSON，也就是二进制 JSON。字段值可以包含其他文档，数组及文档数组。

下文是在Linux 64位系统中安装MongoDB的流程（以下操作以root用户身份进行操作）：

## 2、安装

```shell
# 下载
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz
# 解压
tar -zxvf mongodb-linux-x86_64-3.0.6.tgz
# 将解压包拷贝到指定目录
mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb
```

编写/etc/profile文件，在文件目录添加以下内容

```shell
export PATH=/usr/local/mongodb/bin:$PATH
```

保存之后再执行`source /etc/profile`命令让配置立即生效

## 3、运行

创建数据库目录

```shell
mkdir -p /data/db
```

启动数据库，在启动数据库之前，先确定`/etc/profile`文件的变更已经生效。或者进入bin目录直接执行命令，命令后加”&“进行后台启动，以下是直接使用可执行文件绝对路径启动

```shell
/usr/local/mongodb/bin/mongod &
```

## 4、测试数据库

使用以下命令进入数据库

```shell
/usr/local/mongodb/bin/mongo
```

进入数据库之后，可以使用以下方法关闭数据库服务：

```shell
use admin 
db.shutdownServer()
```

这样就成功安装并测试了，更多内容可以参考官方文档。
