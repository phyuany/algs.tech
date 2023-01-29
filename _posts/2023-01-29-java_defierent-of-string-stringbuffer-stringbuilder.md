---
title: Java中String、StringBufer、StringBuilder的区别以及使用方法
date: 2023-01-29 21:40:00 +0800
categories: [java]
tags: []
pin: false
---

> 撰写时间：2018-09-16，整理时间：2023-01-29。原文链接：<https://www.jb51.net/article/107795.htm>

## 一、简介

在Java语言中，共有8个基本的数据类型，分别为：byte、short、int、long、float、double、boolean 和 char，其中char类型用于表示单个字符，例如 a、b、c 、A、B、C、& 这些大小写字母或者特殊字符等等。在实际的编程中，单个的字符并没有我们想象中用的那么频繁，反而是多个字符组成的“字符串”更为常见，但是在基本的数据类型中，并没有字符串这种数据类型。为了解决这个问题，Java 语言为我们提供了一个被 final 关键字修饰的类 String，她就表示字符串类型，同时由于其被 final 修饰，这也表明咱们只能用这个类创建字符串以及调用其中的方法，却不能继承她。

虽然Java语言为我们提供了字符串类String，能让我们方便的使用字符串类型，姑且这么说，但是在不断的发展中，我们发现单纯的String类型，并不足以满足我们的需求啦！因此，在String类型的基础上，又衍生出了两个字符串构建器StringBuffer和StringBuilder。

## 二、String

我们已经大致了解了String类型啦！说她是一个数据类型，但她并不是基本数据类型，而是一个被final修饰的、不可被继承的类，位于java.lang 包。至于如何使用String 类型，有两种方法，一是直接赋值，二是用new创建，具体示例如下：

```java
// 1、直接赋值
String str1 = "维C果糖"；
// 2、用 new 运算符创建
String str2 = new String("维C果糖")；
```

在常见的字符串操作中，判断两个字符串是否相等尤为常见，且常用的判别方式有两种，即用String类提供的方法`equals`和`==`运算符，下面是使用频率比较高的String类的 API方法：

```java
//* 如果字符串以 suffix 结尾，则返回 true，否则返回 false */
boolean endsWith(String suffix)
/* 如果字符串与 other 相等，则返回 true，否则返回 false */
boolean equals(Object other)
/* 如果字符串与 other 相等（忽略大小写），则返回 true，否则返回 false */
boolean equalsIgnoreCase(String other)
/* 返回字符串的长度 */
int length()
/* 返回一个新字符串，这个字符串用 newString 字符串代替原始字符串中所以的 oldString 字符串，可以用 String 或者 StringBuilder 对象作为 CharSequence 参数 */
String replace(CharSequence oldString, CharSequence newString)
/* 如果字符串以 prefix 开始，则返回 true，否则返回 false */
boolean startsWith(String prefix)
/* 返回一个新字符串，这个字符串包含原始字符串中从 beginIndex 到串尾或 endIndex-1 位置的所以代码单元 */
String substring(int beginIndex)
String substring(int beginIndex, int endIndex)
/* 返回一个新字符串，这个字符串将原始字符串中的所以大写字母都改成了小写字母 */
String toLowerCase()
/* 返回一个新字符串，这个字符串将原始字符串中的所以小写字母都改成了大写字母 */
String toUpperCase()
/* 返回一个新字符串，这个字符串将删除元字符串头部和尾部的空格 */
String trim()
```

### 三、StringBuffer

在我们了解了String类之后，我们会发现她有些缺陷，例如当我们创建了一个String类的对象之后，我们很难对她进行增、删、改的操作。为了解决这个弊端，Java语言就引入了StringBuffer类。StringBuffer和String类似，只是由于StringBuffer的内部实现方式和String不同，StringBuffer在进行字符串处理时，不用生成新的对象，所以在内存的使用上StringBuffer要优于String类。

在StringBuffer类中存在很多和String类一样的方法，这些方法在功能上和String类中的功能是完全一样的。但是有一个非常显著的区别在于，StringBuffer对象每次修改都是修改对象本身，这点是其和String类的最大区别。

此外，StringBuffer是线程安全的，可用于多线程。

> 这里说一下什么是线程安全：
> 一个线程访问资源的时候为其加锁，别的线程只有等到该线程释放资源后才能使用，这样做为了防止数据的非正常改变和使用，这里所说的资源通常是程序中的全局变量

而且StringBuffer对象的初始化与String对象的初始化不大一样，通常情况下，我们使用构造方法进行初始化，即：

```java
// 声明一个空的 StringBuffer 对象
StringBuffer sb = new StringBuffer();
// 声明并初始化 StringBuffer 对象
StringBuffer sb = new StringBuffer("维C果糖");
// 下面的赋值语句是错的，因为 StringBuffer 和 String 是不同的类型
StringBuffer sb = "维C果糖";
// 下面的赋值语句也是错的，因为 StringBuffer 和 String 没有继承关系
StringBuffer sb = (StringBuffer)"维C果糖";
// 将 StringBuffer 对象转化为 String 对象
StringBuffer sb = new StringBuffer("维C果糖");
String str = sb.toString();
```

下面是StringBuffer的常用API方法

```java
/* 构造一个空的字符串构建器 */
StringBuffer()
/* 返回构建器或缓冲器中的代码单元（字符）数量 */
int length()
/* 追加一个字符串并返回一个 this */
StringBuffer append(String str)
/* 追加一个字符并返回一个 this */
StringBuffer append(Char c)
/* 将第 i 个代码单元设置为 c */
void setCharAt(int i, char c)
/* 将构建器的内容进行顺序的反转 c */
StringBuffer reverse()
/* 返回一个与构建器或缓冲器内容相同的字符串 */
String toString()
```

## 四、StringBuilder

在JDK 5.0之后，Java语言又引入了StringBuilder类，这个类的前身是StringBuffer，其效率略微有些低，但允许采用多线程的方式执行添加或者删除字符的操作。如果所有的字符串在一个单线程中（通常都是这样）编辑，则应该用StringBuilder代替她，这两个类的API 是完全相同的。

## 五、总结

过以上的介绍，咱们已经详细的了解了 String、StringBuffer 和 StringBuilder，也知道了她们三个都是用于操作字符串的类。接下来总结一些三者的区别

- 对于操作效率而言，一般来说，StringBuilder > StringBuffer > String；
- 对于线程安全而言，StringBuffer是线程安全的，可用于多线程；而StringBuilder是非线程安全的，用于单线程；
- 对于频繁的字符串操作而言，无论是StringBuffer还是StringBuilder，都优于String。
