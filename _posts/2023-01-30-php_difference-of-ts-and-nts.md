---
title: "PHP TS 和 PHP NTS有什么区别"
date: 2023-01-30 00:20:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2020-12-09，整理时间：2023-01-30。此文参考互联网的文章

## 一、问题

Windows版的PHP从版本5.2.1开始有Thread Safe(线程安全)和None Thread Safe(NTS，非线程安全)之分，这两者不同在于何处？到底应该用哪种？

## 二、分析

从2000年10月20日发布的第一个Windows版的PHP3.0.17开始的都是线程安全的版本，这是由于与Linux/Unix系统是采用多进程的工作方式，不同的是Windows系统是采用多线程的工作方式。在IIS下以CGI方式运行PHP会非常慢，这是由于CGI模式是建立在多进程的基础之上的，而非多线程。

> 公共网关接口（Common Gateway Interface，CGI）是Web 服务器运行时外部程序的规范，按CGI 编写的程序可以扩展服务器功能。

一般我们会把PHP配置成以ISAPI的方式来运行，ISAPI是多线程的方式，这样就快多了。

> ISAPI（Internet Server Application Programming Interface）扩展是可以被 HTTP 服务器加载和调用的 DLL（Dynamic Link Library 即动态连接库）。Internet 服务器扩展也称为 Internet 服务器应用程序 (ISA)，用于增强符合 Internet 服务器 API (ISAPI) 的服务器的功能。

但存在一个问题，很多常用的PHP扩展是以 Linux/Unix的多进程思想来开发的，这些扩展在ISAPI的方式运行时就会出错搞垮IIS。因此在IIS下CGI模式才是PHP运行的最安全方式，但CGI模式对于每个HTTP请求都需要重新加载和卸载整个PHP环境，其消耗是巨大的。

为了兼顾IIS下PHP的效率和安全，微软 给出了FastCGI的解决方案。FastCGI可以让PHP的进程重复利用而不是每一个新的请求就重开一个进程。同时FastCGI也可以允许几个进程同时执行。这样既解决了CGI进程模式消耗太大的问题，又利用上了CGI进程模式不存在线程安全问题的优势。

## 三、总结

因此，如果是使用ISAPI的方式来运行PHP就必须用Thread Safe(线程安全)的版本；而用FastCGI模式运行PHP的话就没有必要用线程安全检查了，用None Thread Safe(NTS，非线程安全)的版本能够更好的提高效率。
