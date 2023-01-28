---
title: MySQL数据库基础的操作与增删改查方法
date: 2023-01-28 15:35:00 +0800
categories: [关系型数据库]
tags: []
pin: false
---

## 一、环境

在前面,我们已经给大家演示了如果安装MySQL，如果你还没有安装MySQL，你可以参考以下相关链接进行安装

文章链接:

- [如何在Windows上安装MySQL 8.0数据库](<https://blog.jkdev.cn/index.php/archives/176/>)
- [在Linux从二进制文件安装MySQL 8.0数据库](<https://blog.jkdev.cn/index.php/archives/125/>)
- [在Linux上从源码安装MySQL 8.0数据库](<https://blog.jkdev.cn/index.php/archives/191/>)

视频教程链接:

- [如何在Windows上安装解压版的MySQL 8.0 数据库](<https://www.bilibili.com/video/av48488984>)
- [如何在阿里云Linux服务器上安装MySQL 8.0 数据库](<https://www.bilibili.com/video/av26079879>)
- [怎样从源码安装MySQL 8.0 数据库](<https://www.bilibili.com/video/av59168411>)

如果你还不会安装MySQL和连接MySQL数据库，你应该选择上面部分内容学习之后，再看以下内容

## 二、概述

实际上，作为一个软件开发者，或者即将成为软件开发者。MySQL数据库或者说数据库，我们需要学习知识很多很多，而此文章，带大家进入MySQL的入门学习，目的是让大家快速学会使用。实际上你在今后学习中遇到的困难，你应该学会使用网络，去查看更多文档。

以下所有命令操作的前提是，添加MySQL安装目录下的bin目录到系统的PATH环境环境变量，如果你还没有做这个步骤，请参考之前的视频教程。在计算机中，bin关键字一般指的是binary单词的缩写，也就是二进制文件的意思，二进制文件就可以说是通过编译后的可执行的文件。

## 二、MySQL服务器基本操作

### 3.1 登录数据库

```shell
mysql -uroot -p;
```

参数说明：
`-u`：在上面示例中-u参数后面紧跟的是`root`关键字，意思就是使用root账号登录
-`p`：此参数代表使用密码登录，加上此参数之后，命令行会提示用户输入MySQL账号对应的密码

默认情况下，使用上面的命令MySQL会连接到安装在本机的MySQ服务。此处说的“本机”指的是你正在使的操作系统，包括Windows、Linux、Mac，或者你的服务器。如果你需要使用本机的MySQL命令去连接远程的MySQL数据库服务器，只需要加一个-h参数即可，代表服务器主机，如下代码

```shell
mysql -uroot -h120.77.41.111 -p;
```

假设`120.77.41.111`是阿里云的一台服务器，在这台服务器安装了MySQL数据库服务，并对外开放了MySQL的服务端口，那么，我们使用以上这条命令即可连接到安装在`120.77.41.111`这台主机上的MySQL。

MySQL默认服务端口是3306，mysql会将3306作为默认服务端口。假设`120.77.41.111`这台服务器在安装MySQL服务的时候把它指定为`3307`，那么在使用mysql命令进行连接时，我们需要使用-P参数指定MySQL的服务端口号，如下代码

```shell
mysql -uroot -h120.77.41.111 -P3307 -p
```

在上面所示例的命令中，你会发现，参数标示和参数是紧紧挨着的，实际上，我们也可以在参数标标识和参数值之间加一个英文输入法状态下的空格，如下代码所示

```shell
mysql -u root -h 120.77.41.111 -P 3307 -p
```

### 3.2 更改账号密码

MySQL 8.0初始化会生成一个默认的密码，并且我们需要更改之后才能使用。在之前的安装MySQL 8.0的文档中，已经给说明如何在MySQL控制台上更改账号的密码和验证方式，在这里不再重复。这里我们使用MySQL服务自带的mysqladmin命令去更改账号的密码。如下代码所示

```shell
mysqladmin -uroot password 'root123456' -p
```

以上代码代表使用mysqladmin命令把root账号的密码改为root123456，后面的-p参数会让系统自动弹出密码输入窗口，此时输入root账号之前的密码按回车之后，新的密码就生效了。

### 3.3 MySQL基本操作的常用命令

登录数据库之后，我们可以调用以下相关命令

(1)查询当前数据库

```sql
show databases;
```

(2)切换某个数据库，如切换到mysql库

```sql
use mysql;
```

(3)查看某个库的所有表名称

```sql
show tables;
```

(4)查看某个表的全部字段

```sql
desc 表名;
```

例如，我们需要查看mysql库的user表，代码如下

```sql
use mysql;
desc user;
```

(5)查看建表语句

```sql
show create table 表名;
```

继(4)，查看user表的建表语句，代码如下

```sql
show create table user;
```

(6)查看当前登录的用户

```sql
select user();
```

(7)查看当前使用的数据库

```sql
select database();
```

(8)新建一个数据库

```sql
create database 数据库名;
```

例如，新建一个db_test数据库，代码如下

```sql
reate database db_test;
```

(9)在某个数据库里新建一张表

```sql
create table 表名(字段名 数据类型,...)
```

继(8)，新建一个user表

```sql
create table user(id int(11), name varchar(45));
```

一般情况下，为了避免字段名与MySQL关键字冲突，在建表过程中，一般给字段加上原意字符，如下代码：

```sql
create table user(`id` int(11), `name` varchar(45));
```

### 3.4 创建一个普通用户并给其授权

创建一个test用户,允许其在任意主机登录，密码为test123

```sql
create user 'test'@'%' identified by 'test123'; 
```

将db_test库的所有权限赋予haha

```sql
grant all on db_test.* to 'haha'@'%';
```

## 四、常用的增删改查语句

### 4.1 查询语句

查询语句代码如下

```sql
select 字段名 from 表名
```

查询db_test库user表的name，代码如下

```sql
use db_test;
select name from user;
```

或者

```sql
select name from db_test.user;
```

在查询所有字段时，用*代替，如下代码

```sql
select * from user;
```

### 4.2 插入一条数据

插入一条数据代码如下

```sql
insert into 表名 values (插入的值);
```

插入一条数据，代码如下

```sql
insert into user values (2,'haha');
```

### 4.3 更改一条数据

更改一条数据代码如下

```sql
update 表名 set 字段名 = 新值 where 条件;
```

把id为2的用户名字改为lisi，代码如下

```sql
update user set `name` = 'lisi' where id = 2;
```

### 4.4 删除一条数据

删除一条数据代码如下

```sql
delete from 表名 where 条件;
```

我们把id为2的用户删除掉，代码如下

```sql
delete from user where id = 2;
```

### 4.5 清空某一表的数据

清空一个表的数据代码如下

```sql
truncate table 表名;
```

我们清空user表，代码如下

```sql
truncate table user;
```

### 4.6 删除某张表

代码如下

```sql
drop table 表名;
```

删除我们创建的user表

```sql
drop table user;
```

## 五、MySQL数据库的备份与恢复

实际上，MySQL给我们提供了备份数据和恢复数据的功能。退出mysql命令行控制台，进入到系统命令控制台中，我们可以使用mysqldump命令对数据库进行备份，还可以用mysql命令对数据库进行恢复。

### 5.1 ＭySQL数据备份

将db_test库备份到db_test.sql文件中，代码如下

```sql
mysqldump -uroot -p db_test > db_test.sql
```

### 5.2 ＭySQL数据恢复

讲db_test.sql数据恢复到db_new数据库中，代码如下

```sql
mysql -uroot -p db_new < db_test.sql
```

## 六、总结

通过此博客，对于初学者来说，学会了MySQL数据库的基本操作，不过，不要高兴的太早，实际上MySQL的知识不仅仅这些。MySQL会长期更新，每次更新都会带来新特性。

作为开发者，我们需要不断学习与巩固，入门学习时间短，但是忘的也快。只有不断重复使用旧知识与学习新知识，我们才能到达熟练。学习本来就是这样的，不是一两天的事，可以说是几个月，或者几年，甚至一辈子。只有坚持才会有收获！
