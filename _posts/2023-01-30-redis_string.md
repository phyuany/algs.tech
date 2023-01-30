---
title: redis入门知识第4篇-redis中的string数据类型与基本的数据存取操作
date: 2023-01-30 23:55:00 +0800
categories: [redis]
tags: []
pin: false
---

## 一、概述

redis 最常应用于各种结构类型和非结构类型高热度数据的访问加速。在本文，我们将从 redis 中 string 数据类型开始了解 redis 对数据的存取操作。本文是该系列的第四篇原创笔记，如果你还没阅读之前的部分，可以通过以下链接进行阅读

- [01-redis入门知识第1篇-redis简介](https://blog.jkdev.cn/index.php/archives/447/)

- [02-redis入门知识第2篇-redis的安装与测试](https://blog.jkdev.cn/index.php/archives/454/)

- [03-redis入门知识第3篇-redis的基本操作与数据类型](https://blog.jkdev.cn/index.php/archives/455/)

## 二、string 类型的特征

- 存储的数据：单个数据，最简单的数据类型，也是最常用的存储类型

- 存储数据的格式：一个存储空间保存一个数据

- 存储空间：通常使用字符串，如果存储的字符是数值的形式，可以使用数值操作（比如增加指定值、减少指定值）的功能

## 三、string 类型的基本操作

- 添加/修改数据

```shell
set key value
```

- 获取数据

```shell
get key
```

- 删除数据

```shell
del key
```

- 添加/修改多个数据（M 即 multiple）

```shell
mset key1 value1 key2 value2 ...
```

- 获取多个数据

```shell
mget key1 key2 ...
```

- 获取数据字符个数

```shell
strlen key
```

- 追加信息到原始数据末尾（如原始存在则追加，否则新建）

如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。

```shell
append key value
```

## 四、单数据操作 与 多数据操作的对比

### 4.1 指令

- 设置单条数据：`set key value`
- 设置多条数据：`mset key1 value1 key2 value2`

### 4.2 操作时间差

假设每执行一个动作需要一个时间单位，那么执行一次命令，需要的时间单位为：发送执行命令（往）的时间 + 执行的时间 + 返回结果（返）的时间，一共需要 3 个时间单位。

- 单数据操作，执行 3 条指令的执行过程：往返 6 个单位 + 执行 3 个

- 多数据操作，执行 3 条指令的执行过程：往返 2 个单位 + 执行 3 个

## 五、string数值类型数据的操作

设置数值数据增加指定的值

```shell
# 对key的值增加1个单位
incr key
# 指定对key的值增加increment个单位
incrby key increment
# 指定对key的值增加increment值，increment可以是float类型的值
incrbyfloat key increment
```

设置数值数据减少指定的值

```shell
# 对key的值减少1个单位
decr key
# 指定对key的值减小increment个单位
decrby key increment
```

string 在 redis 中内部存储默认就是一个字符串，当遇到增减类操作 incr，decr 时会转成数值型进行计算。redis 所有的操作都是原子性的，采用单线程处理所有的业务，命令是一个一个执行的，因此无需考虑并发带来的数据影响。

需要注意的是，按数值进行操作的数据，如果原始数据不能转成数值，或超越 redis 数值的上限范围，将会报错。redis中数值数据最大值为 9223372036854775807（java 中 long 型数据的最大值，Long.MAX_VALUE）

在大型企业级应用中，因为大量的数据，所以通常使用分表的方式存储数据。在使用多张表存储同类型数据中，对应的主键 id 必须保证统一性，不能重复。Oracle 数据库具有 sequence 设定，可以解决该问题，但是 MySQL 数据库并不具有此类机制。那么我们就可以通过 string 数值类型的增加操作获得下一个值，再作为关系数据库中的主键值。

## 六、设置string类型数据的有效期

以下有几个例子

**（1）**：“最强女生”启动海选投票，只能通过微信投票，每个微信号每 4 个小时只能投 1 票。

**（2）**：电商商家开启热门商品推荐，热门商品不能一直处于热门期，每种商品热门期维持 3 天，3 天后自动取消热门。

**（3）**：新闻网站会出现热点新闻，热点新闻最大的特征是实效性，如何自动控制热点新闻的时效性。

- 解决方案：设置数据具有指定的生命周期

```shell
# 以秒为单位设置key的值
setex key seconds value
# 以毫秒为单位设置key的值
psetex key millisenconds value
```

- 查看key有生命周期的数据

```shell
# 以秒为单位查询key剩余的生命周期
ttl key
# 以毫秒为单位查询key剩余的生命周期
pttl key
```

redis 控制数据的生命周期，通过数据是否失效控制业务行为，适用于有所具有时效性限定控制的操作。

## 七、redis 操作反馈

数据类型操作不成功的反馈与数据正常操作的反馈的有查询，如下

- 表示运行结果是否成功

（integer）0 -> false 失败

（integer）1 -> true 成功

- 表示结果值

（integer）3 -> 3 个

（integer）1 -> 1 个

- 表示数据未获取到

（nil）等同于 null

数据最大存储量为512m，而数值计算最大范围（java 中的 long 的最大值）为-9223372036854775807 到 9223372036854775807

## 八、redis中key的命名建议

redis用于缓存热点数据，但数据最终存储在数据库中，redis一般用于关系型数据库中的数据缓存。所以在给缓存key命名时最好要语意化，规范化。如下列 `key -> value` 键值对例子。

- 用户的粉丝数：`user:id:100:fans -> 123355`
- 用户的博客数：`user:id:100:blogs -> 99`
- 用户的关注数：`user:id:100:focus -> 83`

上面的示例是存储用户的单个字段信息，在实际中，我们可以需要存储用户的完成信息，那么中通常以 json 格式存储用户信息，如下 `key -> value` 示例

- 用户的信息：`user:id:100 -> {"id":100,"name":"春晚","fans":12355,"blogs":99,"focus:83}`
