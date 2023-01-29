---
title: "解决执行phpize报错：perl: warning: Falling back to a fallback locale (\"en_US.utf8\")."
date: 2023-01-30 00:08:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2018-03-22，整理时间：2023-01-30

今天在服务器上安装php的curl扩展，但是在运行phpize命令时遇到"perl: warning: Falling back to a fallback locale ("en_US.utf8")."错误提示，如图：

![02.png](/img/php/01-01.png)

根据提示，估计是系统语言库缺失的问题，于是安装i18n(internationalization:国际化)语言包：如下

```shell
apt install locales-all
```

很幸运，再次运行phpize命令，成功解决问题
