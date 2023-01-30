---
title: redis入门知识第9篇-key的通用操作
date: 2023-01-30 23:59:00 +0800
categories: [redis]
tags: []
pin: false
---

## 一、概述

key 是一个字符串，通过 key 获取 redis 中保存的数据，那么 key 通常存在以下的操作

- 对于key自身状态的相关操作，例如：删除、判定是否存在、获取类型 等

- 对于key有效控制的相关操作，例如：有效期设定、判定是否有效、有效状态的切换 等

- 对于key快速查询操作，例如：按指定策略查询key

在本节，我们将介绍 key 的通用操作

## 二、key的基本通用操作

删除指定key

```shell
del key
```

判定key是否存在

```shell
exists key
```

获取 key 的类型

```shell
type key
```

## 三、key的实效性控制操作

- 为指定key设置有效期

```shell
# 设置key有效期为seconds秒
expire key seconds
# 设置key有效期为milliseconds毫秒
pexpire key milliseconds
# 设置key失效 的 秒级时间戳
expireat key timestamp
# 设置key失效的 毫秒级时间戳
pexpireat key milliseconds-timestamp
```

- 获取key的有效时间

```shell
# 获取key的秒级有效时间
ttl key
# 获取key的毫秒级有效时间
pttl key
```

对于获取有效时间的指令，key 不存在返回 -2，key 存在但是没有关联超时时间返回 -1，如果key存在并且有关联时间，则返回具体的剩余时间秒或者毫秒。

- 切换key从实效性转为永久性

```shell
persist key
```

## 四、key的查询操作

key可以使用正则表达式的方式进行查询，查询指令为

```shell
keys pattern
```

以下是常用的查询示例

`*`: 匹配任意数量的任意字符

`?`: 批评为任意一个符号

`[]`: 匹配一个指定符号

```shell
# 查询所有
keys *
# 查询所有以it开头的key
keys it*
# 查询所有以it结尾的key
keys *it
# 查询前面以两个任字符，后面以it结尾的key
keys ??it
# 查询以user:开头，任意一个字符结尾的key
keys user:?
# 查询以u开头，以er:1结尾，中间包含 s 或 t 字符的key
keys keys u[st]er:1
```

## 五、key的其他操作

- 将key改名

```shell
# 当 newkey 已经存在时， rename 命令将覆盖旧值
rename key newkey
# 当且仅当 newkey 不存在时，将 key 改名为 newkey
renamenx key newkey
```

- 排序

对 list, set 或sorted set 中的元素进行排序输出，sort 指令功能比较多，在本文中我们暂且 指演示简单的用户

```shell
# 对list数据倒序输出
sort key desc
# 对lsit数据顺序输出
sort key asc
```

- 查看更多通用操作

```shell
help @generic
```
