---
title: SQL Srver中常用的约束语句
date: 2023-01-28 15:13:00 +0800
categories: [关系型数据库]
tags: []
pin: false
---

> 撰写时间：2017-12-10，整理时间：2023-01-28

## 一、基本概念

SQL中有五类常用的约束，用来对某个字段的数据进行约束：

`not null`：非空约束，指定某列不为空
`unique`：唯一约束，指定某列和几列组合的数据不能重复
`primary key`：主键约束，即非空加上唯一
`foreign key`：外键，指定该列记录属于主表中的一条记录，参照另一条在主表中的数据
`default`：默认值，当添加一条数据记录时，如果没有某一个字段没有插入数据，则自动保存默认数据
`check`：检查，指定一个表达式，用于检验指定数据

## 二、语法

### 2.1 添加主键约束

```sql
alter table 表名
add constraint 约束名 primary key (主键)
```

### 2.2 添加唯一约束

```sql
alter table 表名
add constraint 约束名 unique (字段)
```

### 2.3 添加默认约束

```sql
alter table 表名
add constraint 约束名 default ('默认内容') for 字段
```

### 2.4 添加检查check约束,要求字段只能在1到100之间

```sql
alter table 表名
add constraint 约束名 check (字段 between 1 and 100 )
```

### 2.5 添加外键约束(“主表”和“从表”建立关系)

```sql
alter table 从表
add constraint 约束名 foreign key(关联字段) references 主表(关联字段)
```

### 2.6 查看表中的所有约束

```sql
sp_helpconstraint 表名
```

### 2.7 删除约束

```sql
alter table 表名 drop constraint 约束名
```

## 三、示例

下面以以一个简单的银行数据库中的部分表为例，在建立表的时候添加对应的约束：

```sql
create table branch(
    branch_name varchar(20),
    branch_city varchar(20),
    assert numeric(8,2),
    -- 添加branch表的主键
    constraint branch_primary primary key(branch_name)
)

create table loan(
    loan_number varchar(20),
    branch_name varchar(20),
    amount numeric(8,2),
    -- 添加loan表的主键
    constraint loan_primay primary key (loan_number),
    -- 添加loan表中branch_name的外键，参考branch主表
    constraint loan_branch_name_foreign foreign key (branch_name) references branch
)

-- 删除loan表中的外键约束
alter table loan drop constraint loan_branch_name_foreign
```

本文原创自“极客开发者”，禁止转载！
