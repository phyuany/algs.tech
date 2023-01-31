---
title: SQLite的安装与基本操作方法
date: 2023-01-31 21:31:00 +0800
categories: [工具]
tags: [sqlie]
pin: false
---

## 一、安装

SQLite下载链接: [https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)，此教程基于最新的SQLite3数据库引擎

### 1.1 Windows

从 Windows 区下载预编译的二进制文件。

* 需要下载 **sqlite-tools-win32-\*.zip** 和 **sqlite-dll-win32-\*.zip** 压缩文件。

* 创建文件夹 C:\sqlite，并在此文件夹下解压上面两个压缩文件，将得到 sqlite3.def、sqlite3.dll 和 sqlite3.exe 文件。

* 添加 C:\sqlite 到 PATH 环境变量，最后在命令提示符下，使用 **sqlite3** 命令，将显示如下结果。

### 1.2 Linux

很多Linux都自带SQLite，使用以下命令“sqlite3”命令检测SQLite是否存在，如果不存在，有两种安装方式：

（1）使用从软件库中安装，在debian/ubuntu系统中，可使用以下命令完成安装

```shell
sudo apt install sqlite
```

如果使用apt工具安装，apt会将sqlite2和sqlite3都安装到系统中，使用sqlite和sqlite3区分不同的版本

（2）从源代码进行编译安装

下载源代码**sqlite-autoconf-\*.tar.gz**

安装步骤如下

```shell
tar -zxvf sqlite-autoconf-*.tar.gz
cd sqlite-autoconf-*
./configure --prefix=/usr/local/sqlite
make
make install
```

将SQLite命令工具所在目录添加到系统环境变量中

```shell
sudo vim /etc/profile
```

在文件末尾添加以下代码：

```shell
PATH = /usr/local/sqlite/bin:$PATH
```

## 二、SQLite基本命令

### 2.1 基本操作

（1）进入数据库命令工具

```shell
sqlite3
```

（2）退出命令工具

```shell
.quit
```

### 2.2 创建数据库

```shell
sqlite3 DatabaseName.db
```

SQLite数据库是一个轻量级的数据库系统，数据保存在一个文件中

### 2.3 创建表

（1）语法

用.tables命令查看所有表

```sql
.tables
```

创建表基本语法如下

```sql
CREATE TABLE table_name(
   column1 datatype  PRIMARY KEY(one or more columns),
   column2 datatype,
   column3 datatype,
   .....
   columnN datatype,
);
```

（2示例

下面是一个示例，它创建了一个 user 表，id 作为主键，NOT NULL 的约束表示在表中创建纪录时这些字段不能为 NULL

```sql
CREATE TABLE user(
   id INT PRIMARY KEY     NOT NULL,
   name           TEXT    NOT NULL,
   age            INT     NOT NULL
);
```

### 2.3 删除表

```sql
DROP TABLE table_name;
```

### 2.4 插入一条数据

（1）语法

INSERT语句用于SQLite插入数据，INSERT INTO 语句有两种基本语法，如下

```sql
INSERT INTO TABLE_NAME [(column1, column2, column3,...columnN)]  
VALUES (value1, value2, value3,...valueN);
```

column1、column2、...columnN 是要插入数据的表中的列的名称。如果要为表中的所有列添加值，也可以不需要在 SQLite 查询中指定列名称。但要确保值的顺序与列在表中的顺序一致。

```sql
INSERT INTO TABLE_NAME VALUES (value1,value2,value3,...valueN);
```

（2）示例

下面实现数据插入示例

```sql
INSERT INTO user(id,name,age) VALUES(1,'zhangsan',18);
INSERT INTO user VALUES(2,'lisi',20);
```

### 2.5 查询数据

SQLite 的 **SELECT** 语句用于从 SQLite 数据库表中获取数据，以结果表的形式返回数据。这些结果表也被称为结果集。基本语法如下：

```sql
SELECT column1, column2, columnN FROM table_name;
```

column1、column2...是表的字段。如果想获取所有可用的字段，那么可以使用下面的语法：

```sql
SELECT * FROM table_name;
```
