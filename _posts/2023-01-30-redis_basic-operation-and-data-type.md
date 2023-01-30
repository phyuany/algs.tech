---
title: redis入门知识第3篇-redis的基本操作与数据类型
date: 2023-01-30 23:54:00 +0800
categories: [redis]
tags: []
pin: false
---

## 一、概述

在我的前两篇笔记中，介绍了redis的基本概念，以及安装了redis的学习环境。在这篇文章中，我们一起来熟悉 redis 的基本操作。redis数据存在内存中，可以让程序高效地读取。但它也能将数据写入硬盘内进行永久保存，在这篇文章开始，我们逐渐熟悉redis对数据的操作。

如果你还没阅读过之前的内容，可以通过以下链接阅读前面的部分

- [01-redis入门知识第1篇-redis简介](https://blog.jkdev.cn/index.php/archives/447/)

- [02-redis入门知识第2篇-redis的安装与测试](https://blog.jkdev.cn/index.php/archives/454/)

## 二、redis的基本操作

### 2.1 添加数据

进入redis命令行模式

```shell
./src/redis-cli
```

设置 key、value 数据

- 命令格式

```shell
set key value
```

- 示例

```shell
set name jkdev
```

### 2.2 数据查询

功能：根据 key 查询对应的 value，如果不存在，则返回空 （nil）

- 命令格式

```shell
get key
```

- 示例

```shell
get name
```

### 2.3 清除屏幕信息

- 命令

```shell
clear
```

活着 按`Ctrl` + `L` 也可以清除屏幕信息

### 2.4 查看帮助文档

- 命令格式

```shell
help 命令名称
help @组名
```

- 示例：使用 `help get` 指令获取 `get` 指令的帮助，如下图

![03-01.png](/img/redis/03-01.png)

- 示例：使用 `help @string` 指令获取 `string` 类型的帮助，如下图

![03-02.png](/img/redis/03-02.png)

### 2.5 退出命令行模式

我们可以使用`quit`指令或者`exit`指令，退出cli客户端

## 三、redis的使用场景

redis因为数据存储在内存中，即可提供高性能的数据读取使用，因此通常用于用于数据的缓存。以下是redis的常用场景

### 3.1 原始业务设计

- 秒杀
- 618 活动
- 双 11 活动
- 排队购票

### 3.2 运营平台监控的突发高频访问数据

- 突发的要闻，被强势关注围观

### 3.3 高频、复杂的数据统计

- 在线人数

也就是说，redis可用于数据的高并发场景，在高并发的场景下，程序直接存内存中读取数据。

## 四、redis 数据类型

在redis中，可以存储一下5种数据类型

- string （字符串，类比 java 中 String）
- hash （散列值，类比 java 中 HashMap）
- list （列表，类比 java 中 LinkedList）
- set （集合，类比 java 中 Set）
- sorted_set （有序集合，类比 java 中 TreeSet）

redis 自身是一个映射（map），其中所有的数据都是采用 key:value（键值对） 的形式，
数据类型是指存储的数据类型，也就是 value 的类型，key 永远是字符串。
