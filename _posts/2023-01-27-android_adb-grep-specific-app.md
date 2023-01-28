---
title: Android怎样打印指定包的日志
date: 2023-01-27 23:22:00 +0800
categories: [Android]
tags: [adb]
pin: false
---

> 撰写时间：2022-03-17，整理时间：2023-01-27

先使用ps命令查看报名的进程id

```shell
adb shell ps | grep com.demo.app
```

再根据进程ID过滤日志，加入进程id是 `32182` ，命令如下

```shell
adb logcat | grep 32182
```
