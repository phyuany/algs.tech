---
title: Android开发中包的定义
date: 2023-01-26 01:45:00
categories: [Android]
tags: [规范]
pin: false
---

## 1、概述

Java是一门跨平台的全能面相对象编程语言，在服务端、Android以及桌面软件中都占很大比例，目前，Java也是世界上使用人数最多的编程语言。今天给大家分享Android开发中Java包定义的理解。

## 2、为什么要定义 Java包？

Java具有的开发特点是面相对象，简单的说，Java开发者们在开发程序的时候，可以很好的把模型(Modle)、用户视图(View)、控制器(Controller)等不同功能的代码分开来写，这样不仅便于理解更便于代码的维护。我认为面向对象的核心思想是“使用人类处理问题的方式去开发程序”。我们会根据不同类的功能，区分在不同的包中。

## 3、在Android开发中Java包

在Android应用程序的开发中，根据Android自身的特性以及Java本身的特点，我给大家分享我在Android开发中常定义的包：

- `activity`:  用于保存Activity类
- `service`:  保存服务相关的类
- `util`:  全称utilities（工具），把各种工具类保存在util包内
- `engine`:  引擎包，保存各种驱动类，比如数据库驱动
- `receiver`:  广播接收者包，用于存放广播接收者类
- `view`:  自定义控件包，把自定义的控件都放在这里
- `dao`: （全称：Data Access Objects) 把有关数据库操作的类放在这里
- `bean`:  类似于JavaWeb开发中的JavaBean一样，bean包是专门放置属性类的。比如在数据库中创建了一个表，那么从数据中读取表数据时，就可以先封装一个类，不同的属性对应数据库中的不同字段，并设置get和set函数对属性进行获取和修改。
