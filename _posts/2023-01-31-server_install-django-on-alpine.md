---
title: 怎样在AlpineLinux中搭建Python3+Django环境
date: 2023-01-31 00:13:00 +0800
categories: [服务器]
tags: []
pin: false
---

## 一、概述

Alpine Linux是一个十分轻量级的Linux发行版本，其Docker镜像大概只有5m。现在，我们将从Alpine中构建Python3+Django环境。

演示环境：Alpine 3.11 的Docker容器环境

接下来我们将从一个纯净的Alpine系统开始搭建Python3+Django运行环境。首先在本机的Linux桌面环境开启一个Linux容器，如下代码

```shell
docker run -it --name django -p 80:80 -p 465:465 -p 9090:9090 -p 8001:8001 -p 8002:8002 -v $PWD:/www alpine:3.11
```

## 二、详细过程

### 2.1 切换加速镜像软件源

编辑 `/etc/apk/repositories`，将里面 `dl-cdn.alpinelinux.org` 的改成 `mirrors.aliyun.com` ，保存退出即可
可以直接使用一下命令进行替换

```shell
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
```

### 2.2 安装Python3

Django是一个Django的框架，所以Python是必须的，此次我们安装Python3

```shell
apk add --no-cache python3 python3-dev python3-pip
```

实际上，我们还需要安装python3-dev软件库，否则在安装Django时会报错。同时我们需要安装python3-pip

### 2.3 安装基本基本的开发工具包

我们在开发程序是会使用到一些基本的工具包，我们先安装上，否则在编译或者运行程序时可能会出错

```shell
apk add --no-cache zlib-dev bzip2-dev pcre-dev openssl-dev ncurses-dev sqlite-dev readline-dev tk-dev
```

### 2.4 安装编译工具

后面我们可能会用编译工具编译源代码，我们先安装上基本的编译工具

```shell
apk add --no-cache gcc g++ make cmake
```

### 2.5 安装easy_install

```shell
# 升级pip
pip3 install --upgrade pip
# 安装setuptools
pip3 install setuptools
```

实际上，安装之后会自带easy_install，我在进行测试的时候系统是Python3.8.1版本，即可使用以下命令查看easy_install的版本信息。

```shell
easy_install-3.8 --version
```

### 2.6 安装uwsgi

```shell
apk add --no-cache linux-headers #安装依赖
pip3 install uwsgi
```

测试 uwsgi 是否正常：

新建 test.py 文件，内容如下：

```shell
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    test ="Hello World"
    return test.encode("utf-8")
```

因为我们使用的是Python3，所以需要制定编码，再返回，然后在终端运行以下代码：

```shell
uwsgi --http :8001 --wsgi-file test.py #后台运行
```

此时通过电脑访问<http://127.0.0.1:8001>
如果正常显示"Hello World"，否则检查一下安装过程

### 2.7 安装Django

```shell
pip3 install django
```

检查django是否正常

```shell
django-admin.py startproject demosite
cd demosite
python3 manage.py runserver 0.0.0.0:8002
```

在浏览器访问：<http://127.0.0.1:8002>
检查django是否运行正常。

### 2.8 安装Nginx

```shell
# 下载
wget http://nginx.org/download/nginx-1.17.8.tar.gz
# 解压
tar -zxvf nginx-1.17.8.tar.gz
cd nginx-1.17.8/
# 编译配置
./configure --prefix=/usr/local/nginx \
--with-http_stub_status_module \
--with-http_gzip_static_module
# 编译与安装
make && make install
```

使用一下命令测试是否Nginx正常

```shell
/usr/local/nginx/sbin/nginx
```

然后在浏览器打开<http://127.0.0.1>
如果不正常，检查一下安装过程

### 2.9 配置uwsgi

在/etc/目录下新建uwsgi9090.ini，添加如下配置：

```shell
[uwsgi]
socket = 127.0.0.1:9090
master = true         #主进程
#vhost = true          #多站模式
#no-site = true        #多站模式时不设置入口模块和文件
workers = 2           #子进程数
reload-mercy = 10     
vacuum = true         #退出、重启时清理文件
max-requests = 1000   
limit-as = 512
buffer-size = 30000
pidfile = /var/run/uwsgi9090.pid    #pid文件，用于下面的脚本启动、停止该进程
daemonize = /www/uwsgi9090.log
```

创建对应的日志文件

```shell
touch /www/uwsgi9090.log
```

### 2.10 配置nginx

找到nginx的安装目录（本次安装在：/usr/local/nginx/），打开conf/nginx.conf文件，修改server配置：

```shell
server {
    listen       80;
    server_name  localhost;
    
    location / {            
        include  uwsgi_params;
        uwsgi_pass  127.0.0.1:9090; #必须和uwsgi中的设置一致
        uwsgi_param UWSGI_SCRIPT demosite.wsgi; #入口文件，即wsgi.py相对于项目根目录的位置，“.”相当于一层目录
        uwsgi_param UWSGI_CHDIR /demosite;#项目根目录
        index  index.html index.htm;
        client_max_body_size 35m;
    }
}
```

在浏览器输入：<http://127.0.0.1>，你就可以看到 django 的 OK 了。
