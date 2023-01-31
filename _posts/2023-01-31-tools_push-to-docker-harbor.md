---
title: 如何上传docker镜像到docker hub
date: 2023-01-31 21:11:00 +0800
categories: [工具]
tags: [docker]
pin: false
---

## 1、注册docker hub账号

如果你刚好看到这篇博客，我相信你正在学习Docker基础知识。docker hub就像github一样，是一个托管的服务，不过docker hub托管的是docker镜像，而github托管的是代码。docker hub是docker官方提供的镜像托管服务。

如果没有的docker hub的账号需要先注册一个。地址为[https://hub.docker.com/](https://hub.docker.com/)

### 2、创建一个远程仓库

如图所示，我的账号是`jkdev`，创建一个空的`hello`镜像，`jkdev`是我的仓库名，`hello`是镜像名
![深度截图_20180529174337.png](/img/tools/03-01.png)

## 3、在本地登录docker hub账号

```shell
# 登录docker
service docker start
# 登录docker官方镜像服务
sudo docker login
```

## 4、创建本地镜像

(1)首先查看本地的所有镜像:

```shell
sudo docker images
```

我们基于ubuntu镜像创建了一个hello镜像，如图所示
![深度截图_20180529174830.png](/img/tools/03-02.png)

(2)修改本地镜像tag与远端仓库一样，创建镜像可以指定标签，标签可以理解为镜像的版本，如果不指定标签，它会默认使用最新的版本(latest)

```shell
sudo docker tag <镜像名>:<标签>  <仓库>/<镜像名>:<标签>
```

实际创建的命令如下:

```shell
docker tag hello jkdev/hello
```

我们再次查看本地所有镜像，可以看到多出了一个仓库名和远程仓库一样的镜像，如图所示
![深度截图_20180529181055.png](/img/tools/03-03.png)

(3)推送到远程仓库：

```shell
sudo docker push jkdev/hello
```

一定要注意的是：本地的镜像名称必须和远端一致，否则将无法推送完成。命令执行成功后，我们可以在docker hub控制台看到我们推送的镜像信息。如果我们需要下载下来，使用以下命令即可

```shell
sudo docker pull jkdev/hello
```
