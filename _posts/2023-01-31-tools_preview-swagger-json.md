---
title: 本地查看Swagger导出的Json文件的方法
date: 2023-01-31 21:36:00 +0800
categories: [工具]
tags: []
pin: false
---

## 一、概述

对于Swagger API文档，可以在本地部署Swagger UI用于查看。以下是搭建步骤

## 二、搭建过程

### 2.1 安装npm

后面我们用到的工具都会用到npm，所以先安装npm
安装npm可以参考[https://blog.jkdev.cn/index.php/archives/277/](https://blog.jkdev.cn/index.php/archives/277/)

### 2.2 安装swagger-ui

通过git安装swagger-ui，在安装过程中如果用淘宝镜像可能安装失败，遇到失败的情况可以尝试还原默认的npm镜像

```shell
git clone https://github.com/swagger-api/swagger-ui.git
cd swagger-ui
npm install
npm run dev
```

看到成功提示之后，可以打开在浏览打开器 <http://localhost:3200>，就可以看到本地的swagger-ui服务

### 2.3 安装http-server

http-server可以模拟一个简单的http服务器，我们将swagger文件放在服务器中，用swagger-ui访问http路径的文档

```shell
npm install --global http-server
```

加入我们的文件在`~/Documents/swagger.json`接下来进入到文档目录，并开启http服务

```shell
cd ~/Documents/
http-server --cors
```

`--cors`参数代表能跨域访问

我们就可以在浏览器查看文档了，如下图
![截屏2021-04-07 下午3.31.36.png](/img/tools/13-01.png)
