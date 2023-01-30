---
title: 在Linux桌面下使用adb进行Android投屏
date: 2023-01-31 00:35:00 +0800
categories: [技巧]
tags: [adb]
pin: false
---

先安装上adb和ffplay，投屏命令如下文

- 指定参数运行

```shell
adb shell screenrecord --bit-rate=16m --output-format=h264 --size 540x960 - | ffplay -framerate 60 -framedrop -bufsize 16M -
```

- 使用默认参数运行

```shell
adb shell screenrecord --output-format=h264 - | ffplay -
```
