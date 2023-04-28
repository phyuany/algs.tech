---
title: 怎样创建Android的密钥库
date: 2023-04-29 01:45:00 +0800
categories: [Android]
tags: [keystore]
pin: false
---

## 一、前提

我们需要安装Java，Java将自带keytool工具，使用keytool工具创建密钥库。

## 二、Linux/Unix

在 Linux/Unix 中创建命令如下：

```shell
keytool -genkey -v -keystore ~/.android/jkdev.keystore -alias jkdev -keyalg RSA -keysize 2048 -validity 10000
```

查看命令如下：

```shell
keytool -list -v -keystore ~/.android/jkdev.keystore
```

## 三、windows

在 windows 中创建命令如下：

```shell
keytool -genkey -v -keystore %UserProfile%\.android\jkdev.keystore -alias jkdev -keyalg RSA -keysize 2048 -validity 10000
```

查看命令如下：

```shell
keytool -list -v -keystore %UserProfile%\.android\jkdev.keystore
```
