---
title: 怎样解决在Linux上AndroidStudio连接不上Android设备的问题
date: 2023-01-27 23:19:00 +0800
categories: [Android]
tags: [adb]
pin: false
---

> 撰写时间：2020-02-05，整理时间：2023-01-27

## 一、概述

Arch Linux是一个十分简洁的Linux系统，很多内容是用户自定义的，不像Ubuntu或者Deepin那样开箱即用。所以在使用Arch Linux时出现问题也是正常的。

我安装好AndroidStudio之后，用手机链接上USB，开启开发者模式，不过AndroidStudio开发工具里没有显示设备名称，而是显示一个unkonw device，此时我又把adb命令所在目录添加到PATH环境变量，执行`adb devices`命令之后出现以下错误：

```shell
error: insufficient permissions for device
See [http://developer.android.com/tools/device.html] for more information
```

## 二、分析与解决方案

通过查阅资料得知，这是没有正常驱动Android的原因。以下是解决方案

### 2.1 查看电脑链接的设备

执行lsusb命令，结果如下

```shell
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 002: ID 13d3:56b2 IMC Networks Integrated Camera
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 0cf3:e500 Qualcomm Atheros Communications 
Bus 001 Device 006: ID 0000:3825   USB OPTICAL MOUSE
Bus 001 Device 008: ID 19d2:ffcf ZTE WCDMA Technologies MSM Android
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

可以看到以下这一行，是我手机的关键信息

```shell
Bus 001 Device 008: ID 19d2:ffcf ZTE WCDMA Technologies MSM Android
```

在第6列中":"前是厂商ID：19d2，后是设备ID：ffcf，接下来会用到

### 2.2 添加配置文件并修改权限

```shell
sudo vim /etc/udev/rules.d/51-android.rules
```

添加以下内容

```shell
SUBSYSTEM=="usb",ATTRS{idVendor}=="19d2",ATTRS{idProduct}=="ffcf",MODE="0666"
```

修改权限

```shell
sudo chmod a+rx 51-android.rules
```

### 2.3 重启adb服务

```shell
sudo adb kill-server
sudo adb start-server
```

此时手机出现以下界面

![01.png](/img/android/04-01.png)

勾选确定即可，AndroidStudio就可以正常使用真机调试了，并且adb命令正常使用
