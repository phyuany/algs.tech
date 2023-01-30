---
title: redis入门知识第6篇-list数据类型与基本操作
date: 2023-01-30 23:56:00 +0800
categories: [redis]
tags: []
pin: false
---

## 一、概述

假设我们有这样的需求：我们存储多个数据，并对数据进入存储空间的顺序进行区分。前面介绍的数据类型已经不满足我们现有的需求，于是引入一个新的数据类型 -- list ，list 也可以叫列表， 能保存多个数据，底层使用双向链表存储结构实现（链表属于《数据结构》的归属课程，我们在这里不再赘述）。

![image2-6.png](/img/redis/06-01.png)

本文是该系列文章的第六篇，你可以通过下列链接阅读往期的篇章：

[01-redis入门知识第1篇-redis简介](https://blog.jkdev.cn/index.php/archives/447/)

[02-redis入门知识第2篇-redis的安装与测试](https://blog.jkdev.cn/index.php/archives/454/)

[03-redis入门知识第3篇-redis的基本操作与数据类型](https://blog.jkdev.cn/index.php/archives/455/)

[04-redis入门知识第4篇-redis中的string数据类型与基本的数据存取操作](https://blog.jkdev.cn/index.php/archives/463/)

[05-redis入门知识第5篇-hash数据类型与基本操作](https://blog.jkdev.cn/index.php/archives/465/)

## 二、list 类型数据基本操作

- 添加/修改数据

命令格式

```shell
# 从左边放数据
lpush key value1 [value2] ...
# 从右边放数据
rpush key value1 [value2] ...
```

示例，往右边 添加 a b c 三个数据到 list 键中

```shell
RPUSH list a b c
```

- 查询数据

命令格式

```shell
# 根据索引范围查询数据
lrange key start stop
# 根据索引查询数据
lindex key index
# 查看list的长度
llen key
```

示例，查询第一个元素： list 中索引从 0 开始到 1 的元素

```shell
LRANGE list 0 1
```

查询所有元素： list 中索引从 0 开始到 倒数第 1 的元素

```shell
LRANGE list 0 -1
```

- 获取并移除数据

命令格式

```shell
# 从左边出数据
lpop key
# 从右边出数据
rpop key
```

## 三、list 类型数据扩展操作

- 规定时间内获取并移除数据

命令格式

```shell
# 阻塞式从list左边取数据
blpop key1 [key2] ... timeout
# 阻塞式从list右边取数据
brpop key1 [key2] ... timeout
```

示例，从key1 列表中，或 key2 列表中，或 key3 列表中，阻塞式 取出列表数据，阻塞等待时间是 200 秒。

下面这条指令的功能是：从三个列表中任意取一个数据数据，从key1 开始取，如果 key1 没有数据则从 key2 取， key2 没有则从 key3 取。如果都没有 redis 会等待 200 秒，如果在 200 秒内没有取到，将会 返回 `nil`

```shell
BRPOP key1 key2 key2 200
```

- 移除指定数据

应用案例： 微信朋友圈点赞，要求按照点赞顺序显示点赞好友信息。如果取消点赞，移除对应的好友信息。这就就用到我们将说的，移除指定数据。

指令格式

```shell
# 移除指定个数的数据
lrem key count value
```

示例

```shell
# 添加9个数据到001列表中
RPUSH 001 a b c d e d e f g
# 移除1个”d“
LREM 001 1 d
# 移除2个”e“
LREM 001 2 e
```

最后通过`LRANGE 001 0 -1`命令查询结果如下

```shell
1) "a"
2) "b"
3) "c"
4) "d"
5) "f"
6) "g"
```

总结：我们可以使用 list 的特性，可以将 redis 应用于操作具有先后顺序的数据控制

## 四、redis数据注意事项

- list 中保存的数据都是 string 类型，数据总量是有限的，最多 2 的 32 次方减 1 个元素（4294967295）

- list 具有索引的概念，但是操作数据时通常以队列的形式进行入队出队操作，或者栈的形式进行入栈出栈操作

- 获取全部数据操作结束索引设置为-1

- list 可以对数据进行分页操作，通常第一页的信息来自 list，第 2 页及更多的信息通过数据库的形式加载
