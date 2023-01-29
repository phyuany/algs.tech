---
title: "怎样解决LayUI提示Uncaught TypeError: Cannot create property 'LAY_TABLE_INDEX' on string '20'错误"
date: 2023-01-29 20:35:00 +0800
categories: [前端]
tags: []
pin: false
---

> 撰写时间：2020-02-28，整理时间：2023-01-29

## 一、概述

我在项目后台使用LayUI作为后台管理界面，展示列表的时候前端出现了如题所示错误，截图如下

![01.png](/img/frontend/04-01.png)

实际上，在后端代码我使用ThinkPHP 5.0框架，即使你使用语言或者其他框架时，也不妨碍你对以下内容的理解。

## 二、解决过程

ThinkPHP 5.0有一个分页查询的函数。我直接将查询到的结果集返回。因此报错。

其实，在概述的代码报错的原因是：layUI要求data字段返回数组格式的数据。否则layUI解析出现错误，我们更改后端代码之后。返回数据如下

![03.png](/img/frontend/04-02.png)

这时候layUI解析没有错误，layUI列表正常显示了。
