---
title: CSS的基本使用方法与学习工具推荐
date: 2023-01-29 20:40:00 +0800
categories: [前端]
tags: []
pin: false
---

> 撰写时间：2020-12-07，整理时间：2023-01-29

## 一、概念

CSS是层叠样式表（Cascading Style Sheets），用来描述文档呈现的样式语言。这里所说的文档通常是HTML文档。在学习CSS之前，应当具备基础的HTML知识。

## 二、引入CSS样式

### 2.1 引入外部css样式文件

通常在文档头部使用 link 标签引入外部定义文件，如下

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="jkdev.css">
</head>
<body>
    
</body>
</html>
```

完整的定义应该写上文档类型

```html
<link rel="stylesheet" href="jkdev.css" type="text/css">
```

但默认情况下，不写也没事，浏览器会给我们做兼容，也能正常解析。

### 2.2 内部定义CSS样式

我们还可以把CSS定义的代码和HTML混合在一页，编写，通常在文档头部加上style标签，并在style标签内定义样式

让h1标签内文字变成红色，可以这样写

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="jkdev.css" type="text/css">
    <style>
        h1{
            color: red;
        }
    </style>
</head>
<body>
    <h1>jkdev.cn</h1>
</body>
</html>
```

### 2.3 行内定义CSS样式

我们也可以直接在对应标签内写使用style属性定义 CSS样式，如下代码让h1标签现实黄色，并且将文字大小改为 12px

```html
<h1 style="color: yellow; font-size: 12px">jkdev.cn</h1>
```

### 2.4 在CSS中引入其他CSS文件定义的样式

我们还可以@import标识在一个CSS样式文件中引入另一个CSS样式文件，在jkdev.css中引入common.css

```css
@import url("common.css");

h1 {
    color: red;
}
```

在文档头部style标签中引入common.css文件

```html
<style>
    @import url("common.css");

    h1{
        color: red;
    }
</style>
```

### 2.5 总结

在CSS定义代码中，可以添加空白行，不影响解析，但是实际中，应当减少空白行的存在，以达到压缩文件的效果。CSS定义的每一条样式定义的条目称为一个表达式，以英文分号为结束标识（“;”），CSS可以使用 `/* */` 进行注释，出现错误样式定义浏览器也不会进行解析。

在jkdev.css文件中定义如下代码

```css
h1{
    /* 设置文字为红色 */
    color: red;
    /* 以下一行为错误语法，将不会被解析 */
    fontsize: 16px;
}
```

浏览器在解析HTML文档时，会从上往下解析，所以CSS定义应当在文档的头部，避免在加载网页的过程中，由于网络的延迟原因，没有及时加载CSS定义文件导致的界面样式错乱。

## 三、安装VsCode的CSS学习插件

### 3.1 Easy Less：添加LESS的支持

LESS是一种CSS语法格式，LESS支持嵌套定义CSS样式，我们可以使用LESS插件，使用LESS语法编写，同时自动生成CSS原生文件。编写jkdev.less 自动生成 jkdev.css

jkdev.less文件内容

```css
div {
    font-size: 12px;

    h1 {
        color: red;
    }
}
```

自动生成的jkdev.css文件

```css
div {
  font-size: 12px;
}
div h1 {
  color: red;
}

```

### 3.2 Live Server：浏览器同步刷新

编辑器在没有安装插件的情况下，我们编辑好的HTML手动刷新浏览器才能预览效果。打开Live Server的模式下，浏览器会自动同步界面的变化。我们可以使用一个窗口编辑代码，另一个窗口打开浏览器查看效果，避免高频率地刷新网页。
