---
title: 怎样搭建golang的开发环境
date: 2023-01-29 21:18:00 +0800
categories: [golang]
tags: []
pin: false
---

> 撰写时间：2020-12-07，整理时间：2023-01-29

## 一、安装go语言

go语言支持多平台的操作系统

go的官网地址为：[https://golang.org](https://golang.org)

国内地址为：[https://golang.google.cn](https://golang.google.cn)

下载对应的安装包，接下来进行安装

### 1.1 UNIX/Linux 安装

以下是使用二进制安装文件的安装方式

```shell
# 解压安装包
tar -zxvf go1.16.5.linux-arm64.tar.gz
# 将安装包移到系统目录
sudo mv go /usr/local
```

设置环境变量，使用`sudo vim /etc/profile`打开系统profile文件，在文件末尾添加以下内容

```shell
# 这是go的安装路径，后续IDE会读取GO_ROOT的内容
export GOROOT=/usr/local/go
# 设置go的国内包加速镜像
export GOPROXY=https://goproxy.cn
# 将go的二进制可执行文件加入PATH环境变量中，即可在终端调用
export PATH=$GOROOT/bin:$PATH
```

最后，执行`source /etc/profile`命令系统重新加载profile文件

我们还可以使用`go nev`命令查看golang所有依赖的环境变量，我们在后面需要使用的时候会逐个介绍。

### 1.2 mac和windows 安装

windows使用`.msi`后缀的文件双击进行安装，mac可以使用1中的二进制包安装方式，也可以使用`.pkg`后缀文件双击进行安装。最后，正确设置`GOROOT`、`GOPROXY`、`PATH` 三个环境变量即可。

### 1.3 安装验证

最后使用`go version`在命令行终端验证，如果如下go的版本信息，则安装成功

```shell
go version go1.16.4 darwin/amd64
```

## 二、配置开发环境

### 2.1 安装IDE

为了方便写go程序，我们需要安装IDE。我们以VS Code为例，配置go的开发环境。

VS code的官网是：[https://code.visualstudio.com/](https://code.visualstudio.com/)

下载我们电脑对应的操作系统，双击根据提示安装即可。

最后，我们打开vs code 的插件商店安装go语言官方的扩展包，如下图

![01-1.png](/img/golang/01-01.png)

### 2.2 编写第一个程序

在任意工作目录，创建`test.go`文件，添加以下内容

```go
package main

import "fmt"

func main(){
  fmt.Println("Hello World")
}
```

在命令行终端使用go命令执行输入结果如下

```shell
pan@pandeMBP learn % go run test.go
Hello World
pan@pandeMBP learn %
```
