---
title: 怎样让docker容器开机自启？
date: 2023-01-31 00:37:00 +0800
categories: [技巧]
tags: []
pin: false
---

首先让docker服务开机自启，如下命令

```shell
systemctl enable docker
```

再让docker 容器开机自启，命令如下

```shell
docker update --restart=always mysql
```
