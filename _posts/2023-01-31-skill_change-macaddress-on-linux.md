---
title: 怎样在Linux中修改网卡MAC地址
date: 2023-01-31 00:37:00 +0800
categories: [技巧]
tags: []
pin: false
---

在Linux系统中，修改网卡MAC地址的方法如下

```shell
# 关闭网卡
/sbin/ifconfig eth0 down
# 修改网卡MAC地址
/sbin/ifconfig eth0 hw ether 00:0C:29:36:97:20
# 开启网卡
/sbin/ifconfig eth0 up
```
