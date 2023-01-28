---
title: 怎样理解MySQL中的字符集和校对集?
date: 2023-01-28 15:15:00 +0800
categories: [关系型数据库]
tags: []
pin: false
---

> 撰写时间：2017-12-14，整理时间：2023-01-28

## 1. 概述

无论Web网页还是手机APP，通常会用到数据库，我们最常用的就是MySQL，对于个人开发者来说，MySQL开源免费、简单快捷。那么我们在创建一个MySQL数据库的时候声明的字符集和校对集是什么？它有什么用？

假如我们需要创建一个用户数据库，我们通常会使用以下代码:

```sql
CREATE DATABASE IF NOT EXISTS `user` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 2. 什么是字符集和校对集

以下是个人理解，如有疑问和异议欢迎留言评论：

### 2.1.字符集

我们都知道计算机底层保存数据都是以二进制代码来进行保存的，实际上就是`0`或者`1`，`default charset`关键字就声明了`user`库默认使用`utf8`编码规则来编码的，也就是说我们`user`库中的数据会以`utf8`编码规则来保存。至于什么是`utf8`，我们理解它就是一种通用的编码规则。

### 2.2.校对集

MySQL中在创建数据库时使用`collate`关键字来声明校对集，从字面意思来看，就是用来比较的。声明`utf8_general_ci`校对集之后数据库系统就会使用`utf8_general_ci`这种校对方式对数据进行比较。例如：我们查询的数据需要根据用户名进行排序，如下代码：

```sql
select * from user order by username;
```

那么数据库系统就会根据`utf8_general_ci`这种校对规则根据`username`字段进行数据比较排序。
