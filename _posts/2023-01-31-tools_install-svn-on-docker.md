---
title: 怎样使用Docker镜像搭建SVN服务
date: 2023-01-31 21:28:00 +0800
categories: [工具]
tags: []
pin: false
---

## 一、概述

### 1.基础环境

- Debian GNU/Linux 9.9 (stretch)
- Docker version 19.03.4

如果你使用其他Linux发行版本安装过程也基本一样的，我在操作的时候使用阿里云的Debian 9.9服务器。关于什么是SVN，这里不会赘述，本文将演示基于第三方镜像搭建SVN服务的过程

### 2.开启简单的容器实例

本次我们直接使用Dockerhub上比较热门的一个镜像`elleflorio/svn-server`，详细内容可以参考此链接:[https://hub.docker.com/r/elleflorio/svn-server](https://hub.docker.com/r/elleflorio/svn-server)

使用以下命令创建一个简单的svn服务

```shell
docker run -d --name svn-server -p 3690:3690 elleflorio/svn-server
```

你还可以选择性的把本地目录映射到容器的svn仓库目录，如下参数

```shell
-v <hostpath>:/home/svn
```

## 二、实际操作过程

假设现在有这样的需要，开发一个PHP项目，在服务器创建一个base代码仓库，所有开发人员可以从服务器把代码pull到PC进行开发，再push到服务器进行托管，开发任务push代码完成之后，将代码同步到同一台服务器的web目录，下面我们基于Docker完成这个过程。

### 2.1 下载镜像，创建容器

```shell
# 下载镜像
docker pull elleflorio/svn-server
# 创建svn仓库目录，进入svn仓库目录
mkdir -p /var/svn
# 创建svn服务容器，把容器中的svn仓库映射到本机，并映射3690端口
docker run -d --name svn-server -p 3690:3690 -v "$PWD":/home/svn -v /var/www/html:/var/www/html elleflorio/svn-server
```

在以上示例代码中，为了能在svn容器中管理本机的项目目录，假设本机的项目目录是`/var/www/html`，除了映射SVN仓库目录，我们同时把本机的`/var/www/html`映射到svn容器中的`/var/www/html`目录。

### 2.2 在服务器创建代码仓库

```shell
# 创建代码仓库
docker exec -t svn-server svnadmin create /home/svn/test
```

以上代码中，在容器中的`/home/svn/test`目录创建代码仓库，会同步到本机的/var/svn目录。我们先进行svn仓库配置。

SVN库中的配置目录 conf 有三个文件:

- authz：权限控制文件
- passwd：帐号密码文件
- svnserve.conf：SVN服务综合配置文件

修改权限配置文件`/var/svn/test/conf/authz`，如下

```conf
[groups]            
# 用户组
admin = master,admin  
#用户组所对应的目录
[/]                 
# 库目录权限
@admin = rw         
# 用户组权限
*=r               
```

配置账号密码文件 passwd，`/var/svn/test/conf/passwd`

```conf
[users]
# harry = harryssecret
# sally = sallyssecret
master = master
admin = admin
```

配置SVN的服务配置文件`/var/svn/test/conf/svnserve.conf`，如下

```conf
[general]
# force-username-case = none
# 匿名访问的权限 可以是read、write，none，默认为read，现改为none
anon-access = none
# 使授权用户有写权限
auth-access = write
# 密码数据库的路径
password-db = passwd
# 授权配置文件
authz-db = authz
# 认证命名空间，SVN会在认证提示里显示，并且作为凭证缓存的关键字
realm = /var/svn/test
[sasl]
```

完成以上配置之后，我们要做的是服务器代码与本地代码同步，如下图所示：
![4053052780.png](/img/tools/09-01.png)

下面，我们将本机电脑代码推送到SVN仓库后，服务器又将代码同步到项目目录。

### 2.3 同步代码到服务器项目目录

- 在PC上将代码上传到svn仓库（PC上必须安装SVN）

```shell
# 将仓库中的代码pull到本地,下面的123.123.123.123代表的是服务器IP地址，以下过程可能会需要输入svn账号和密码
svn checkout svn://123.123.123.123/test
# 进入代码目录
cd test
# 创建示例文件
echo "hello" >> test.txt
# 提交代码到SVN仓库
svn add test.txt
svn commit test.txt -m 'test'
```

- 服务器同步代码

```shell
# 使用checkout指令将代码同步到项目目录
docker exec -t svn-server svn checkout svn://127.0.0.1/test /var/www/html/test --username master --password master --force --no-auth-cache
```

- 自动同步代码

我们开发的PC每一次向服务器提交一次代码，现在都需要去服务器主动执行一下同步命令，代码才会同步到服务器的项目目录。想必很麻烦，于是我们可以使用SVN提供的钩子去实现代码自动更新，如下配置

```shell
# 进入钩子配置文件目录
cd /var/svn/test/hooks
# 复制钩子模板文件为钩子文件
cp post-commit.tmpl post-commit
```

接下来我们编辑post-commit文件，注释掉发送邮件的代码。然后在下面加上两行代码即可，如下：

```shell
REPOS="$1"
REV="$2"
TXN_NAME="$3"

#mailer.py commit "$REPOS" "$REV" /path/to/mailer.conf
# 设置编码
export LANG="en_US.UTF-8"
# 更新代码到项目目录
svn update --username master --password master /var/www/html/test
```

编辑完成配置文件之后，我们只需要保存即可，自动生效。此时在PC上修改项目内容，再次提交到代码仓库，服务器会自动同步到服务器的项目目录。不再需要手动执行svn checkout指令

## 三、总结

本次主要演示了SVN服务器的搭建过程中的一些基本流程。我们使用第三方Docker镜像来构建SVN容器服务，实际上和我们直接在操作系统上手动编译安装，或者从软件库安装的效果一样。而容器更好的减少服务器相关依赖，也更好地隔离操作系统的环境。如果我们不需要SVN服务了，直接将对应容器移除即可。

如果我们每次创建一个SVN仓库，都要进行一大堆配置，是一件比较浪费时间的事。开发者也不应该把过多的时间与精力去“重复造轮子”，而应该把更多的时间和精力去思考更有价值的事物！大家可以自行学习编写一个Shell脚本，用于快速搭建SVN仓库，就可以使用一条命令即可完成以上复杂的内容。限于篇幅，本文不再深入研究。
