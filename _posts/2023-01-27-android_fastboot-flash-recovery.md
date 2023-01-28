---
title: 使用Google官方的fastboot工具刷入第三方recovery
date: 2023-01-27 23:16:00 +0800
categories: [Android]
tags: [fastboot]
pin: false
---

> 撰写时间：2018-06-01，整理时间：2023-01-27

## 一、fastboot简介

如果你正在阅读这篇博客，相信你是一位Android发烧友。如果你对Android刷机还不太了解的话，以下文章你可能看不懂，也欢迎留言评论。fastboot是Android SDK中的一个小工具，当然它很小，你也可以从很多网上论坛找到。我们怎样利用fastboot工具刷入第三方recovery程序呢。以下是具体步骤。

## 二、测试工具

(1) Nubia z9 mini　手机
(2) fastboot 工具
(3) twrp recovery (从官网下载)

我直接使用Android SDK里面的，如果你电脑没有Android SDK，你可以自己去网上找一个对应你系统的fastboot，下载到本地后，记得把fastboot所在路径加入path环境变量

## 三、刷入过程

(1) 手机连接电脑，安装对应的Android驱动程序
(2) 手机在关机状态下同时长按 音量-、电源 先进入到fastboot模式
(3) 执行刷写命令

```shell
fastboot flash recovery 对应的img包
```

另外一种是，先执行了命令，命令行终端会进入一个等待状态，手机重启到fastboot模式后立即刷入，如下图：

![深度截图_20180601112013.png](/img/android/03-01.png)

再次进入recovery模式，已经刷入新的recovery了!
