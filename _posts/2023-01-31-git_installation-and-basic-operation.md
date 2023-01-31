---
title: 使用git托管代码的基本操作方法
date: 2023-01-31 21:54:00 +0800
categories: [git]
tags: []
pin: false
---

> 我计划写2到3篇的git基础文章，这次介绍 git 的基本使用方法。后面介绍企业中如何使用 git 进行协同开发。

## 一、Git 简介

### 1.1 架构

关键词：`git` 或 `Git`

Git 是一个分布式的代码托管工具，我们可以基于同一套代码，在不同电脑上进行项目开发，最终都可以可以把代码同步到 Git 服务器上。Git是一个分布式的代码托管工具，如下图：

![01.png](/img/git/01-01.png)

### 1.2 工作流

下面是一张基于 Git 托管的代码流向图

![02.png](/img/git/01-02.png)

Git涉及到一下关键的几个概念：

**Working Directory (工作区)**：我们编辑与变动的代码文件，都在工作区下；

**Stating Area (缓存区)**：代码有变动的时候，我们可以把每次变动后的代码，提交到Git缓存区。让 Git 记录代码的变动；

**Respository (仓库)**：在Git缓存区的代码，都可以提交到 Git仓库进行托管。从图中可以看到，本地的仓库（一般指我们自己的电脑）可以将代码提交到远程的仓库（一般指Git服务器）。

## 二、安装

### 2.1 Linux

在 Debian、Ubuntu、Deepin 等操作系统上安装命令如下

```shell
sudo apt install git
```

### 2.2 Mac

Mac 可以使用`homebrew`进行安装，如下命令

```shell
# 安装homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# 安装git
brew install git
```

或者安装 [xcode 工具包](https://developer.apple.com/xcode/)，`xcode`工具包包含了`git`，要注意的是，`xcode` 安装时间可能比较长，推荐在网络环境很好的前提下使用这种方式。

### 2.3 windows

windows 安装比较简单，直接使用二进制包按提示安装即可，git二进制安装包下载地址在 [Git官网上](https://git-scm.com/downloads) 可以找到。

## 三、基本使用

### 3.1 初始化本地用户

在使用git之前，我们先设置本地的账号，设置一下本地 `git`用户 的用户名和邮箱，如下命令

```shell
# 配置用户名
git config --global user.name "John Doe"
# 配置邮箱
git config --global user.email johndoe@example.com
```

配置好本地账号信息之后，就可以使用了。实际上本地的账号可以任意设置，与远端账号关系不大，只是记录我们在本地仓库操作的用户信息而已。

### 3.2 创建本地仓库

我们在命令行终端上进入到自己的工作目录，假如自己的工作目录在 `/home/pan/work/src` ，我们先进入这个目录，再执行git初始化命令即可，如下命令

```shell
# 进入工作目录
cd /home/pan/work/src
# 初始化git仓库
git init
```

初始化完成后当前目录变成了 `git` 的工作目录，此时在这个目录下会生成名为 `.git` 的隐藏目录，这个目录就是 `git` 保存文件变动信息的目录，本地的所有变动记录都在这里。这个目录不能删除，如果删除之后，工作目录将不再是一个 `git` 的工作目录。

### 3.3 将本地代码提交到Git缓存区

我们可以在本地添加一个代码文件，如下命令

```shell
# 创建一个c++源代码文件
touch test.cpp
```

这个时候，我们可以使用git 将 `test.cpp` 源代码文件提交到 `git` 缓存区，使用以下命令

```shell
# 将 test.cpp 文件提交到 git 缓存区
git add test.cpp
```

或者使用另一个命令

```shell
# 将当前目录所有文件提交到 git 缓存区
git add .
```

### 3.4 将缓存区的代码提交到本地仓库

使用下面命令将代码提交到本地仓库，就完成托管了，如下命令

```shell
# 将本地git缓存区代码提交到本地仓库，-m 参数后面是提交备注
git commit -m "first commit"
```

提交到git仓库的代码，我们就可以使用 `git` 的很多实用功能，如回退代码、查看代码变动历史等等。

### 3.5 将代码提交到远程仓库

本地仓库的可以提交到远程仓库，远程仓库也可以实时同步最新代码到本地仓库。在实际开发中，一般是多个人共用一个远程仓库，因为一个稍微有规模的项目通常不是一个人开发的，而是多个人共同维护一套代码。下面是将本地代码同步到远端的基本操作：

假如我们在Github仓库上创建了一个仓库，仓库的git地址为 `https://github.com/kotlindev/test.git`,我们使用以下命令即可将本地代码提交到远端仓库。

```shell
# 将本地仓库绑定远程仓库
git remote add origin https://github.com/kotlindev/test.git
# 将本地的当前master分支代码提交待远端master
git push -u origin master
```

进行账号验证之后，即可成功将代码推送到远端git仓库。

## 四、总结

通过这篇文章，我们已经会将自己的代码推送到远端的 Github 进行托管了，但实际上，远远不能满足我们的需求。你可能会有以下疑问：

- 怎么同步远端的最新代码？
- 怎么回退到旧的代码？
- 怎么进行多人协同开发？
- 企业的代码协同过程是怎样的？

......

篇幅问题，后面我们会解决上述我们提到的问题，并循序渐进地介绍怎样git在实际工作的提高我们工作效率。欢迎大家继续关注！
