---
title: 怎样检测SSL证书和密钥是否匹配
date: 2023-01-31 00:51:00 +0800
categories: [技巧]
tags: [vue]
pin: false
---

密钥文件通常是`.key`后缀，证书文件格式通常是`.crt`后缀，检测命令如下

```shell
openssl x509 -noout -modulus -in dl.discuz.chat.crt | openssl md5
openssl rsa -noout -modulus -in dl.discuz.chat.key | openssl md5
```
