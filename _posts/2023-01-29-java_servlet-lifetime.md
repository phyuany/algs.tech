---
title: Servlet的生命周期
date: 2023-01-29 21:34:00 +0800
categories: [java]
tags: []
pin: false
---

> 撰写时间：2018-01-13，整理时间：2023-01-29

Servlet重要的生命周期方法如下：

- 1.构造方法：创建Servlet对象的时候调用。第一次访问Servelet的时候创建Servlet对象。

- 2.init方法：创建完Servlet对象的时候调用。

- 3.service方法：客户端每次发出请求时调用。

- 4.destory方法：销毁servlet对象的时候调用。停止服务器获取重新部署web应用时会调用。

如以下代码：

```java
package servlet;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyServlet extends HttpServlet {

    public MyServlet() {
        System.out.println("构造方法被调用");
    }

    @Override
    public void init() throws ServletException {
        System.out.println("init方法被调用");
    }

    @Override
    protected void service(HttpServletRequest arg0, HttpServletResponse arg1) throws ServletException, IOException {
        System.out.println("service方法被调用");
    }

    @Override
    public void destroy() {
        System.out.println("destroy方法被调用");
    }
}
```
