---
title: "解决java.lang.IllegalArgumentException: An invalid character [32] was present in the Cookie value错误"
date: 2023-01-29 21:35:00 +0800
categories: [java]
tags: []
pin: false
---

> 撰写时间：2018-03-17，整理时间：2023-01-29

## 一、问题描述

最近在学习Java中的Servlet，今天使用Eclipse编写一个Cookie的一个示例，模拟网站回显用户上次访问时间，代码如下：

```java
package cn.jkdev;

import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/HistServlet")
public class HistServlet extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");

        // 获取当前时间
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        String curTime = format.format(new Date());

        // 取得cookie
        Cookie[] cookies = request.getCookies();
        String lastTime = null;
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (cookie.getName().equals("lastTime")) {
                    // 第n次访问
                    // 有lastTime的cookie
                    lastTime = cookie.getValue();// 上次访问的时间
                    // 1.把上次显示时间显示到浏览器
                    response.getWriter().write("欢迎回来，你上次访问的时间为：" + lastTime + ",当前时间为：" + curTime);
                    // 2.更新cookie
                    cookie.setValue(curTime.trim());
                    cookie.setMaxAge(3600);
                    // 3.把更新后的cookie发送到浏览器
                    response.addCookie(cookie);
                    break;
                }
            }
        }

        /**
        * 第一次访问（没有cookie 或 有cookie，但没有名为lastTime的cookie）
        */
        if (cookies == null || lastTime == null) {
            // 1.显示当前时间到浏览器
            response.getWriter().write("你是首次访问本网站，当前时间为：" + curTime);
            // 2.创建Cookie对象
            Cookie cookie = new Cookie("lastTime", curTime);
            cookie.setMaxAge(3600);
            // 3.把cookie发送到浏览器保存
            response.addCookie(cookie);
        }
    }
}
```

当我在浏览器测试，多次点击后，依然如图下图显示效果

![01](/img/java/02-01.png)

## 二、发现问题

通过查看控制台输出信息，发现以下错误：

```java
java.lang.IllegalArgumentException: An invalid character [32] was present in the Cookie value
```

原来如此，Cookie的值是非法的，非法字符是character [32]，于是我找到对应的ASCII码表，如下图，character [32]对应的是一个空格，这样是不可以的。

![ASCII.jpg](/ig/java/02-02.jpeg)

## 三、解决办法

- 1.在构造Date字符串的时候不要使用空格，使用其他字符来替换，这种方法比较简单。
- 2.如果非要使用空格来隔开日期与时间，我们需要对其进行编码就可以了，但是读取的时候不要忘记了解码，不然在用户界面上会出现乱码。

(1)对Date字符串进行编码：

```java
String curTime = URLEncoder.encode(format.format(new Date()), "utf-8");
```

(2)读取的时候进行解码：

```java
response.getWriter().write("欢迎回来，你上次访问的时间为：" + URLDecoder.decode(lastTime, "utf-8") + ",当前时间为：" + URLDecoder.decode(curTime, "utf-8"));
```

通过上面的解决办法，再刷新浏览器两次，正常显示想要的效果的：
