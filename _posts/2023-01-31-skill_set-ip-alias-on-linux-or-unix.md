---
title: 怎样给网卡的IP地址设置别名
date: 2023-01-31 00:49:00 +0800
categories: [技巧]
tags: [vue]
pin: false
---

命令如下，适用于Linux和Mac

```shell
# 设置IP别名
ifconfig en1 alias 192.168.2.1 netmask 255.255.255.0
# 删除IP别名
ifconfig en1 –alias 192.168.2.1 netmask 255.255.255.0
```
