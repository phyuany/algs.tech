---
title: 从线性连续存储开始，重新认识《数据结构》
author: jiker
date: 2023-01-26 13:28:00 +0800
categories: [数据结构与算法]
tags: []
pin: true
---

> 撰写时间：2019-06-13，整理时间：2023-01-26

## 一、数据结构概述

### 1.1 简介

《数据结构》这门课程大多时候等同于《数据结构与算法》，所以我们一般说数据结构，都会涉及到算法。《数据结构》这门课程要求学生根据所学的数据结构理论，能完成复杂的程序设计。而程序设计能力的提高，必须要有学习、观摩、借鉴和实践的过程。

在阅读这篇文章时，你应当具备一定的C/C++编程基础，并能够理解指针和结构体。

### 1.2 数据结构与算法的概念

我们把现实中复杂的问题以特定的数据类型（现实中的个体）和特定的存储结构（现实中个体之间的关系）保存在计算机内存中，我们把这种“特定的存储结构”叫做数据结构。在此数据结构的基础之上为实现的某个功能（比如查找、删除、排序）而执行的相应操作叫做算法。

> 我们可以简单理解为：数据结构 = 个体 + 个体的关系，算法 = 对存储数据的操作。

### 1.3 衡量算法的标准

通过1，我们把处理数据的方法和步骤叫做算法。而衡量算法的标准包括以下几个方面：

- 时间复杂度：程序大概要执行的次数，而非执行的时间
- 空间复杂度：算法执行过程中，大概占用的最大的内存
- 难易程度
- 健壮性

> 算法最核心的内容是研究算法的时间复杂度和空间复杂度。

### 1.4 数据结构在程序开发中的地位

数据结构是软件工程中最核心的课程。在实际程序开发中，我们会使用各种编程语言，对各种数据进行相应的功能操作，如存储、查询、删除，或是更复杂的运算。所以数据结构功底在一定程度上，决定了个人在程序开发上的综合能力。

> 我们可以理解为：程序 = 数据的存储 + 数据的操作（也就是算法） + 可以被计算机执行的语言。

### 1.5 数据结构的基本模块

为了方便我们今后的学习，本文列出数据结构的基础知识模块如下

线性结构（我们可以把它叫做线性表）

- 连续存储[数组]
- 离散存储[链表]
- 线性结构的两种常见应用之一------栈
- 线性结构的两种常见应用之二------队列

非线性结构

- 树
- 图

## 二、连续存储的数据结构与基本算法

> 在数据结构中，我们研究的对象就是数据，其次是研究操作数据的方法与步骤。今天，我们从线性结构中的连续存储开始，从代码的角度，重新认识数据结构。

连续存储实际上就是连续的存储结构，我们可以理解数组就是连续存储的实现。接下来我们通过C语言的```struct```关键字，定义连续存储这种数据结构体，这里我们把它叫做数组，并研究其基本的算法。

连续存储结构容易实现元素的追和读取线性表中的第 i 个元素的操作；但在实现插入、删除操作时，需要移动大量元素。所以，它适合存储相对稳定的线性表，如职工工资表、学生学籍表。

### 2.1 定义数组

首先我们定义一个```01-Arr.cpp```文件，需要引入基本的c语言头文件，如下代码

```c
# include <stdio.h> // 标准io头部，包含printf函数
# include <malloc.h> // 包含malloc函数，在mac电脑上，改为sys/malloc.h
# include <stdlib.h> // 包含exit函数
```

接下来我们定义数组的结构体，该数据类型包含三个成员，pBase、len、cnt，如下代码

```c
struct Arr
{
    int *pBase; // 存储的是数组第一个元素的地址
    int len;    // 数组所能容纳的最大元素个数
    int cnt;    // 当前数组有效元素个数
};
```

### 2.2 初始化数组

接下来我们定义初始化数组的函数，根据数组长度动态分配内存，用于存储对应的数组元素，如下代码

```c
void init_arr(struct Arr *pArr, int length)
{
    // 根据数组的长度，分配的内存pBase
    pArr->pBase = (int *)malloc(sizeof(int) * length);
    if (NULL == pArr->pBase)
    {
        printf("动态分配内存失败");
        exit(-1);
    }
    // 内存分配成功
    pArr->len = length;
    pArr->cnt = 0;
    return;
}
```

### 2.3 判定数组是否为空或是否已满

我们可以根据当前数组的有效元素和数组总元素判定数组是否为空或者是否已满，定义以下两个函数

```c
// 判断数组是否为空
bool is_empty(struct Arr *pArr)
{
    // 数组的有效个数为0，即数组为空
    if (0 == pArr->cnt)
    {
        return true;
    }
    // 否则，数组不为空
    else
    {
        return false;
    }
}
// 判断数组是否已满
bool is_full(struct Arr *pArr)
{
    // 数组的有效个数为数组的长度，即数组已满
    if (pArr->cnt == pArr->len)
    {
        return true;
    }
    // 否则，数组未满
    else
    {
        return false;
    }
}
```

### 2.4 遍历数组元素

遍历数组的目的是把每一个有效的数组元素打印到终端。根据数组的当前有效个数，遍历并打印每个元素，我们定义如下代码

```c
void traverse_arr(struct Arr *pArr)
{
    for (int i = 0; i < pArr->cnt; i++)
    {
        printf("%d\t", pArr->pBase[i]);
    }
    printf("\n");
}
```

### 2.5 追加数组元素

追加即在未满的数组中，在末尾添加元素，如下代码

```c
bool append_arr(struct Arr *pArr, int val)
{
    // 数组已满则返回false
    if (is_full(pArr))
    {
        return false;
    }
    // 数组不满时进行追加
    pArr->pBase[pArr->cnt] = val;
    (pArr->cnt)++;
    return true;
}
```

### 2.6 插入元素

数组除了追加，还有数组插入操作，插入即在特定的位置添加数据，如下代码

```c
bool insert_arr(struct Arr *pArr, int pos, int val)
{
    // 如果数组元素已满，返回false
    if (is_full(pArr))
    {
        return false;
    }
    // 如果要插入的位置不对，返回false
    if (pos < 0 || pos > pArr->cnt)
    {
        return false;
    }
    // 从最后一个元素的位置开始，到要插入的位置，挨个将数组元素往后移一位
    for (int i = pArr->cnt - 1; i >= pos; --i)
    {
        pArr->pBase[i + 1] = pArr->pBase[i];
    }
    // 将插入的值，赋在指定的位置
    pArr->pBase[pos] = val;
    // 标记数组有效元素的字段加1
    (pArr->cnt)++;
    return true;
}
```

### 2.7 删除元素

删除操作和插入操作类似，区别是：插入数据时，从插入位置开始，后面的数据往后移，并在插入的位置加入元素；删除数据时，我们需要在删除位置的后一位开始，后面的数据都往前移，删除位置的数据会被覆盖，如下代码

```c
bool delete_arr(struct Arr *pArr, int pos, int *pVal)
{
    // 如果数组元素已满，返回false
    if (is_full(pArr))
    {
        return false;
    }
    // 如果要插入的位置不对，返回false
    if (pos < 0 || pos > pArr->cnt)
    {
        return false;
    }
    // 将删除的元素的值赋给传进来的指针变量
    *pVal = pArr->pBase[pos];
    // 从到要删除的位置开始，一直到最后一个位置，将数组元素挨个往前移动
    for (int i = pos; i < pArr->cnt; ++i)
    {
        pArr->pBase[i] = pArr->pBase[i + 1];
    }
    pArr->cnt--;
    return true;
}
```

### 2.8 数组倒置

数组倒置的操作是将数组的第1位和最后1位元素进行替换，第2位和倒数第2位元素进行替换，依此类推。最终完成倒置，如下代码

```c
void inverse_arr(struct Arr *pArr)
{
    int i = 0;
    int j = pArr->cnt - 1;
    int t;
    while (i < j)
    {
        t = pArr->pBase[i];
        pArr->pBase[i] = pArr->pBase[j];
        pArr->pBase[j] = t;
        ++i;
        --j;
    }
    return;
}
```

### 2.9 数组排序

接下来是一个简单的排序过程，函数中有两层循环。第一层循环，选择当前位置的元素（从索引为0的位置开始，即未循环前，当前位置是第1个元素），在第二层循环中，从当前位置的元素的下一个元素开始，直到最后一个元素，挨个与当前位置的元素进行大小对比，每次对比都把小的元素替换到当前位置。

第1轮循环结束后，索引为0的位置将是最小的元素，第2轮循环结束后，索引为1的位置将是次小的元素，依此类推。最终，数组元素将按从小到大的顺序排列，如下代码

```c
void sort_arr(struct Arr *pArr)
{
    int i, j, t;
    for (i = 0; i < pArr->cnt; i++)
    {
        for (j = i + 1; j < pArr->cnt; j++)
        {
            if (pArr->pBase[i] > pArr->pBase[j])
            {
                t = pArr->pBase[i];
                pArr->pBase[i] = pArr->pBase[j];
                pArr->pBase[j] = t;
            }
        }
    }
}
```

## 三、测试与验证

最终所有测试代码如下

```c
#include <stdio.h>
#include <sys/malloc.h> // windows中用 #include <malloc.h>
#include <stdlib.h>

// 定义数组结构体
struct Arr
{
    int *pBase; // 存储的是数组第一个元素的地址
    int len;    // 数组所能容纳的最大元素个数
    int cnt;    // 当前数组有效元素个数
};

// 初始化数组
void init_arr(struct Arr *pArr, int length)
{
    // 将分配的内存赋给pBase
    pArr->pBase = (int *)malloc(sizeof(int) * length);
    if (NULL == pArr->pBase)
    {
        printf("动态分配内存失败");
        exit(-1);
    }
    // 内存分配成功
    pArr->len = length;
    pArr->cnt = 0;
    return;
}

// 判断数组是否为空
bool is_empty(struct Arr *pArr)
{
    // 数组的有效个数为0，即数组为空
    if (0 == pArr->cnt)
    {
        return true;
    }
    // 否则，数组不为空
    else
    {
        return false;
    }
}

// 判断数组元素是否已满
bool is_full(struct Arr *pArr)
{
    // 数组的有效个数为数组的长度，即数组已满
    if (pArr->cnt == pArr->len)
    {
        return true;
    }
    // 否则，数组未满
    else
    {
        return false;
    }
}

// 遍历数组
void traverse_arr(struct Arr *pArr)
{
    for (int i = 0; i < pArr->cnt; i++)
    {
        printf("%d\t", pArr->pBase[i]);
    }
    printf("\n");
}

// 追加数组（元素）
bool append_arr(struct Arr *pArr, int val)
{
    // 数组满是返回false
    if (is_full(pArr))
    {
        return false;
    }
    // 数组不满时追加
    pArr->pBase[pArr->cnt] = val;
    (pArr->cnt)++;
    return true;
}

// 插入数组（元素）
bool insert_arr(struct Arr *pArr, int pos, int val)
{
    // 如果数组元素已满，返回false
    if (is_full(pArr))
    {
        return false;
    }
    // 如果要插入的位置不对，返回false
    if (pos < 0 || pos > pArr->cnt)
    {
        return false;
    }
    // 从最后一个元素的位置开始，到要插入的位置，挨个将数组元素往后移一位
    for (int i = pArr->cnt - 1; i >= pos; --i)
    {
        pArr->pBase[i + 1] = pArr->pBase[i];
    }
    // 将插入的值，赋在指定的位置
    pArr->pBase[pos] = val;
    // 标记数组有效元素的字段加1
    (pArr->cnt)++;
    return true;
}

// 删除数组（元素）
bool delete_arr(struct Arr *pArr, int pos, int *pVal)
{
    // 如果数组元素已满，返回false
    if (is_full(pArr))
    {
        return false;
    }
    // 如果要插入的位置不对，返回false
    if (pos < 0 || pos > pArr->cnt)
    {
        return false;
    }
    // 将删除的元素的值赋给传进来的指针变量
    *pVal = pArr->pBase[pos];
    // 从到要删除的位置开始，一直到最后一个位置，将数组元素挨个往前移动
    for (int i = pos; i < pArr->cnt; ++i)
    {
        pArr->pBase[i] = pArr->pBase[i + 1];
    }
    pArr->cnt--;
    return true;
}

// 倒置数组（元素）
void inverse_arr(struct Arr *pArr)
{
    int i = 0;
    int j = pArr->cnt - 1;
    int t;
    while (i < j)
    {
        t = pArr->pBase[i];
        pArr->pBase[i] = pArr->pBase[j];
        pArr->pBase[j] = t;
        ++i;
        --j;
    }
    return;
}

// 数组排序
void sort_arr(struct Arr *pArr)
{
    int i, j, t;
    for (i = 0; i < pArr->cnt; i++)
    {
        for (j = i + 1; j < pArr->cnt; j++)
        {
            if (pArr->pBase[i] > pArr->pBase[j])
            {
                t = pArr->pBase[i];
                pArr->pBase[i] = pArr->pBase[j];
                pArr->pBase[j] = t;
            }
        }
    }
}

// 主函数测试
int main()
{
    struct Arr arr;
    // 初始化长度为6的数组
    init_arr(&arr, 6);

    // 追加元素
    append_arr(&arr, 1);
    append_arr(&arr, 2);
    append_arr(&arr, 3);
    append_arr(&arr, 4);

    traverse_arr(&arr); // 输出结果为 1 2 3 4

    // 在索引为0的位置插入6
    insert_arr(&arr, 0, 5);
    traverse_arr(&arr); // 输出结果为 5 1 2 3 4

    // 在索引为0的位置删除元素
    int delval; // 用于接收删除的元素
    delete_arr(&arr, 0, &delval);
    traverse_arr(&arr); // 输出结果为 1 2 3 4

    // 数组倒置
    inverse_arr(&arr);
    traverse_arr(&arr); // 输出结果为 4 3 2 1

    // 数组排序
    sort_arr(&arr);
    traverse_arr(&arr); // 输出结果为 1 2 3 4

    return 0;
}
```

> 本文原创首发自wx订阅号：**极客开发中**，禁止转载
