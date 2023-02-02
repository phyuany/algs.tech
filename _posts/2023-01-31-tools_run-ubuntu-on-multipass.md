---
title: 怎样使用Multipass快速创建和管理Ubuntu虚拟机？
date: 2023-01-31 21:40:00 +0800
categories: [工具]
tags: [虚拟机]
pin: false
---

> multipass是Ubuntu官方提供管理Ubuntu Server虚拟机的桌面工具，本文将介绍怎样使用multipass搭建Ubuntu Server虚拟机。multipass可以帮助我们快速创建和管理Ubuntu Server虚拟机。

## 一、安装

multipass的官方网站是[https://multipass.run/](https://multipass.run/)， 我们可以下载Linux/Windows/Mac版本。选择对应的版本进行安装。

## 二、使用

### 2.1 查看命令帮助

查看帮助

```shell
multipass --help
# 或
multipass -h
```

查看某个launch指令的帮助

```shell
multipass launch --help
# 或
multipass launch -h
```

### 2.2 创建实例

使用最新LTS镜像创建实例

```shell
multipass launch --name ubuntu
```

查看可用的镜像

```shell
multipass find
```

使用指定LTS版本镜像创建实例

```shell
multipass launch --name ubuntu1 18.04
```

### 2.3 操作实例

进入实例的shell终端

```shell
# 如果对应实例没有运行的话，会主动运行对应实例
multipass shell ubuntu
```

执行实例的shell命令

```shell
# 查看实例版本，如果实例没有在运行，将会提示错误
multipass exec ubuntu -- lsb_release -a
```

### 2.4 查看和管理实例

查看虚拟机实例列表

```shell
multipass list
```

停止和启动实例

```shell
# 停止ubuntu和ubuntu1
multipass stop ubuntu ubuntu1
# 启动ubuntu
multipass start ubuntu
```

清空不需要的实例

```shell
multipass delete ubuntu1
multipass purge
```

使用--all对操作所有实例

```shell
# 启动所有实例
multipass start --all
# 停止所有实例
multipass stop --all
# 删除所有实例
multipass delete --all
```
