---
title: 在Linux上使用二进制安装包安装MySQL8.0
date: 2023-01-28 15:25:00 +0800
categories: [关系型数据库]
tags: []
pin: false
---

> 撰写时间：2018-06-30 14:48，整理时间：2023-01-28

## 一、环境

继MySQL 5.7之后，直接跳到了MySQL 8.0，官方说这次来了个大升级，其他的不说，就查询速度是5.7的2倍，因此我也尝试安装使用，我的Linux版本是 Ubuntu 16.04，根据官方文档，下面是安装的过程

## 二、安装过程

### 2.1 下载安装包

MySQL最新下载地址：[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)
选择的是Linux 64位通用的二级制版本，这样不在需要进行编译安装，系统安装依赖库后就可以直接使用。

### 2.2 安装依赖库

官方说要安装libaio，但实际如果你安装libaio库的话不行，还需安装numactl库，如下

```shell
apt install numactl
apt install libaio-dev
```

### 2.3 解压软件包到系统

解压之后将软件包移动到系统中的/usr/local目录，并命名为mysql

```shell
tar -zxvf mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz
mv mysql-8.0.11-linux-glibc2.12-x86_64 /usr/local/mysql
```

### 2.4 添加用户、设置权限

```shell
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
cd /usr/local
cd mysql
mkdir mysql-files
chown mysql:mysql mysql-files
chmod 750 mysql-files
```

### 2.5 初始化数据库

```shell
bin/mysqld --initialize --user=mysql
```

可以看到系统随机给root用户分配了一个密码，如图所示，这个密码要记住，想要自定义后续可以改

### 2.6 安装SSL服务

执行安装命令之前先安装openssl，不然会报错

```shell
apt install openssl
bin/mysql_ssl_rsa_setup
```

### 2.7 复制服务文件

```shell
cp support-files/mysql.server /etc/init.d/mysql.server
```

## 三、使用MySQL 8.0.11

### 3.1 开启服务

&是后台运行的意思，执行命令之后，终端会卡在一个位置，再按一下Enter即可，如图所示

```shell
bin/mysqld_safe --user=mysql &
```

### 3.2 使用用户root登录

使用刚才随机生成的密码，即可计入数据库

```shell
bin/mysql -uroot -p
```

### 3.3 更改root用户密码

第一次使用随机登录并不能使用，因此我们需要更改密码，如下

- (1)方案一：修改本地登录密码

```shell
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
flush privileges;
```

- (2)方案二：修改密码，并设置为任意IP与第三方客户端可登录

```shell
# 修改root的密码与加密方式
ALTER USER 'root'@'localhost' IDENTIFIED BY '密码' PASSWORD EXPIRE NEVER;
# 切换到mysql库
use mysql;
# 更改可以登录的IP为任意IP
update user set host='%' where user = 'root';
# 刷新权限
flush privileges;
# 再次更改root用户密码，使其可以在任意IP访问
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码';
# 刷新权限
flush privileges;
```

更改好之后，退出，并使用新密码重新登录，数据库便可以使用了！
