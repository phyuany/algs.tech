---
title: 怎样使用Docker运行php项目？
date: 2023-01-28 14:47:00 +0800
categories: [容器与虚拟化]
tags: [docker]
pin: false
---

> 撰写时间：2019-12-14，整理时间：2023-01-28

## 一.概述

Docker就是现在最热门的容器引擎技术，在本文不再赘述基本概念，如果你听说过Docker，那你肯定会听过 "镜像"、"容器" 之类的关键词。不过，这篇博客使用一个简单的示例，给大家演示Docker怎么快速搭建PHP运行环境。

## 二.基础环境

- Linux服务器 (CentOS , Ubuntu , Debian 等, 这里使用的是腾讯云服务器,  预装 Debian 9.0 操作系统)
- Docker

## 三.操作流程

### 3.1 安装Docker

docker公司提供了Docker EE（Enterprise Edition） 和 Docker CE （Community Edition）版本，EE 版本涉及商业服务，而CE版本是Docker的免费产品，包含了完整的Docker平台。

我们可以从各个Linux发行版本的软件库中安装，也可以直接从官方提供的脚本安装，这样能安装最新的版本。因为在国内，我们可以直接通过阿里云镜像安装，使用以下命令

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

安装完成之后我们可以使用以下命令查看Docker版本信息

```shell
docker -v
```

我们可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器,添加以下代码

```json
{
  "registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"]
}
```

重启Docker服务后，我们下载的官方镜像会从阿里云上去拉取，不过私人镜像还是从Docker官方的Dockerhub去拉取，Debian中使用以下代码重启Docker服务

```shell
systemctl restart docker
```

### 3.2 下载Docker镜像

目前比较热门的服务器是 Apache 和 Nginx，实际上，他们各自有优缺点, 不能说谁好谁坏，适合自己的才是最好的。今天我们基于PHP的官方Docker镜像 php-apache 来搭建apache服务器基础环境. 详细文档可以参考[https://hub.docker.com/_/php](https://hub.docker.com/_/php)

不过Dockerhub上官方的php-apache镜像扩展很少，可能满足实际项目中最基本的依赖需求，因此在 php:7.2-apache官方镜像的基础之上我安装了 gd 库(用来处理图片),并安装了 mysqli 和 pdo_mysql扩展，用来驱动 mysql数据库。详细内容可以参考我的个人dockerhub仓库链接 [https://hub.docker.com/r/jkdev/php](https://hub.docker.com/r/jkdev/php)

我们直接使用以下命令拉取自定义的镜像

```shell
# 拉取镜像
docker pull jkdev/php:7.2-apache
# 拉取完成之后,查看本地镜像
docker images
```

### 3.3 创建Docker容器运行项目

我们在服务器上创建一个专门存放web项目的目录,如下代码

```shell
# 创建目录
mkdir /www
# 进入目录
cd /www
```

现在我们基于jkdev/php:7.2-apache镜像创建Docker容器，并把/www目录映射到docker中对应的apache的web项目目录，如以下命令

```shell
docker run -d -p 80:80 -p 443:443 -p 465:465 --name apache -v /etc/localtime:/etc/localtime:ro -v "$PWD":/var/www/html jkdev/php:7.2-apache
```

参数说明

- run: 代表运行一个容器
- -d: 在后台运行容器
- -p: 将宿主机端口与容器端口进行映射,格式为  < 宿主机端口>:<容器端口>
- --name: 指定容器的名称
- -v: 将主机的目录与容器目录进行映射,格式为 <主机目录>:<容器目录>

### 3.4 运行项目

我们使用的镜像中,apache集成了php环境,所以基于此镜像启动的容器可以作为html代码和php代码的容器,并且向公网开放. 下面我们从github上拉取一个html简单项目,作为部署的网站

```shell
# 首先确定我们所在的目录是前面指定web目录
cd /www
# 更新软件仓库
apt update
# 安装git 
apt install git
# 从github拉去一份开源代码
git clone https://github.com/kotlindev/HTML-News-Page.git
# 将源代码复制到web根目录
mv HTML-News-Page/* ./
```

好了，现在我们打开浏览器，就可以看到我们部署的HTML静态界面了

## 四、总结

实际上，这篇博客并不是系统地介绍dcoker基础知识，只是使用简单演示一下docker的一个使用场景，apache服务器也不仅仅这些知识，我们也能感受到docker部署项目的方便与快捷。

后面我将继续使用文章的形式，更系统地介绍docker的基础知识。欢迎持续关注
