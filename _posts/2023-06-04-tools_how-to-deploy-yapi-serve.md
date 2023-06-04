---
title: 怎样使用yapi搭建自己的接口文档平台
date: 2023-05-03 12:13:00 +0800
categories: [工具]
tags: [yapi]
pin: false
---

## 一、概述

yapi是一个开源的接口文档平台，可以用于管理接口文档，同时可以进行接口测试。本文将介绍如何使用yapi搭建自己的接口文档平台。其开源地址为：<https://github.com/YMFE/yapi>。

以下是我本次的运行环境：

- 环境：云轻量服务器
- 操作系统： `Debian 11`

## 二、准备docker环境

### 1.1 安装docker

我们将在docker中运行yapi，所以需要安装docker，安装命令如下：

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

以上命令是使用阿里云的镜像源安装docker，因为在国内，使用阿里云的镜像源会快很多。如果不想使用阿里云的镜像源，直接去掉`--mirror Aliyun`即可。

### 1.2 安装docker-compose

docker-compose是docker的一个单机编排工具，用于管理多个容器。这是`docker-compose`的官网文档链接：<https://docs.docker.com/compose/>。

本次我们使用独立安装的方式安装`dcoker-compose`，安装命令如下：

```shell
# 下载docker-compose二进制安装文件
curl -SL https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

赋予`docker-compose`可执行权限

```shell
chmod +x /usr/local/bin/docker-compose
```

为了检测`docker-compose`是否安装成功，可以执行如下命令：

```shell
docker-compose
```

如果安装之后执行`docker-compose`命令没有报错，说明安装成功。否则请检查安装过程是否有问题。你也可以在`/usr/bin`目录创建软链接。

```shell
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

我们安装好之后，`docker-compose`命令等同于`docker compose`命令。其区别在于，`docker-compose`命令代表它是独立安装的，而`docker compose`命令是docker的子命令。

## 三、安装yapi

我们选择服务器的一个空目录，用于存放yapi的配置文件和数据文件。这里我选择的目录是`/data/yapi`，在下文我把这个目录叫做“工作目录”。

### 3.1 定义yapi的Dockerfile文件

在工作目录下创建`Dockerfile`文件，内容如下：

```dockerfile
FROM node:12-alpine

RUN npm install -g yapi-cli --registry https://registry.npmmirror.com

EXPOSE 3000 9090
```

以上文件是用于构建yapi的docker镜像，其中`node:12-alpine`是yapi的运行环境，`yapi-cli`是yapi的命令行工具，`EXPOSE`是yapi的端口。其中`9090`端口是yapi的安装程序的服务端口，`3000`端口是yapi的服务端口。

### 3.2 定义yapi的docker-compose.yml文件

在工作目录下创建`docker-compose.yml`文件，内容如下：

```yml
version: '3.1'

services:
  mongo:
    image: mongo
    restart: always
    volumes: 
        - ./mongo/data/db:/data/db
    ports: 
        - 27017:27017
    container_name: mongo
    healthcheck:
      test: ["CMD", "netstat -anp | grep 27017"]
      interval: 2m
      timeout: 10s
      retries: 3
  yapi:
    build:
      context: ./
      dockerfile: Dockerfile
    image: yapi
    # first run
    command: "yapi server"
    # after first run
    # command: "node /my-yapi/vendors/server/app.js"
    volumes: 
        - ./my-yapi:/my-yapi
    ports: 
      - 9090:9090
      - 3000:3000
    container_name: yapi
    depends_on: 
      - mongo
```

在以上配置文件中，即使容器被删除，数据也不会丢失，这是因为我们使用了`volumes`配置项。如下：

- `mongo`服务的`volumes`配置项，是用于持久化`mongo`的数据
- `yapi`服务的`volumes`配置项，是用于持久化`yapi`的数据

### 3.3 启动yapi安装服务

在工作目录下执行如下命令：

```shell
docker-compose up -d
```

这个命令会启动`mongo`和`yapi`两个容器，其中`mongo`容器是用于存储`yapi`的数据，`yapi`容器是用于安装`yapi`的。服务正常启动之后，我们在浏览器中访问`http://<服务器IP>:9090`，即可看到`yapi`的安装界面。

进入安装界面之后，我选择最新的版本进行安装，填写配置如下图

![yapi安装界面](/img/tools/23060401.png)

填好配置之后，点击`开始部署`按钮，即可开始安装。安装完成之后，会提示成功安装的管理员信息，我们需要将这些信息保存下来，以便后续登录使用。默认情况下，管理员的账号是我们配置的邮箱，密码是`ymfe.org`。

如果在安装过程中出现错误，可以通过`docker-compose logs`命令查看日志。

### 3.4 启动yapi服务

安装完成之后，我们需要启动`yapi`服务，我们先销毁现有的容器，如下命令

```shell
docker-compose down
```

修改`docker-compose.yml`文件，将`yapi`服务的`command`配置项修改为如下：

```yml
command: "node /my-yapi/vendors/server/app.js"
```

然后执行如下命令启动`yapi`服务

```shell
docker-compose up -d
```

## 四、使用注意事项

### 4.1 浏览器跨域请求插件

在浏览器中访问`http://<服务器IP>:3000`，即可看到`yapi`的登录界面。需要注意的是，如果我们需要在浏览器使用`yapi`请求接口，我们需要在浏览器中安装`yapi`的跨域请求插件，地址如下：

- **Chrome**: <https://chrome.google.com/webstore/detail/yapi-x/ebiododddhjccikhminneafpoppneknc>
- **Edge**: <https://microsoftedge.microsoft.com/addons/detail/crossrequest/bephiepmhphdlafkfonngafenjhfehlb>

如果你在安装的时候，发现这两个链接失效了，可以直接在浏览器的插件商店中搜索`yapi`，即可找到对应的插件。或者通过源码的方式进行安装，此处不再赘述。源码地址为: <https://github.com/YMFE/cross-request>

### 4.2 开启邮件服务和关闭注册功能

我们可以通过`yapi`的配置文件，开启邮件服务和关闭注册功能。我们需要在工作目录下的`my-yapi`目录修改`config.json`文件，内容如下：

```json
{
   "port": "3000",
   "adminAccount": "phy.xyz@foxmail.com",
   "closeRegister": true,
   "db": {
      "servername": "mongo",
      "DATABASE": "yapi",
      "port": "27017"
   },
   "mail": {
      "enable": true,
      "host": "smtp.qq.com",
      "port": 465,
      "from": "QQ邮箱地址",
      "auth": {
         "user": "QQ邮箱地址",
         "pass": "QQ邮箱密码"
      }
   }
}
```

其中，`closeRegister`配置项是用于关闭注册功能，`mail`配置项是用于邮件服务。

### 4.3 使用nginx反向代理

推荐使用Nginx反向代理，可以让我们在浏览器中直接访问`http://域名`，而不是`http://<服务器IP>:3000`。我们需要创建一个`nginx`的配置文件，内容如下：

```nginx
server {
        listen      80;
        server_name yapi.jkdev.cn;

        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        
        location / {
            proxy_pass http://127.0.0.1:3000/;
        }
}
```

以上配置文件是用于将`http://yapi.jkdev.cn`反向代理到服务器的`3000`端口。我们需要将这个配置文件放到nginx的vhost配置文件目录，一般在`/etc/nginx/conf.d`目录下，然后执行`nginx -s reload`命令，即可生效。
