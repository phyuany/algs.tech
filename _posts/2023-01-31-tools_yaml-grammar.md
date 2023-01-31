---
title: YAML的基本语法
date: 2023-01-31 21:24:00 +0800
categories: [工具]
tags: []
pin: false
---

## 一、YAML基本语法

- 使用锁紧表示层级关系
- 锁紧时不允许使用tab键，只允许使用空格
- 大小写敏感觉

`k: v`：表示一对键值对（空格必须有）

以空格的锁紧来控制层级关系，只要是左对齐的一列数据，都是一个层级的，如下示例

```yaml
server:
    port: 8081
    path: /hello
```

## 二、YAML支持的三种数据结构

- 对象：键值对的集合
- 数组：一组按次序排列的值
- 字面量：单个的、不可再分的值

## 三、值的写法

### 3.1 字面量：普通的值（数字，字符串，布尔）

- `K: v`

字面直接来写，字符默认不用加上单引号或者双引号；

- ""

双引号，不会转义字符串里的特殊字符，特殊字符作为本身想表达的意思，`name: "zhangsan \n lisi"`，输出为：zhangsan 换行 lisi

- ''

单引号，会转义特殊字符，特殊字符最终只是一个普通的字符串数据，`name: 'zhangsan \n lisi'`，输出为：zhangsan \n lisi

## 四、对象、Map（属性和值）（键值对）

- 对象

缩进写法

```yaml
friend: 
    lastName: zhangsan
    age: 20
```

行内写法

```yaml
friend: {lastName: zhangsan, age: 18}
```

## 五、数组（List、Set）

用-表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法：

```yaml
pets: [cat,dog,pig]
```
