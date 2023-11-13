---
title: 如何解决双系统中Linux访问不了windows的NTFS分区的问题
date: 2023-01-28 00:15:00 +0800
categories: [操作系统与网络]
tags: [双系统]
pin: false
---

> 撰写时间：2018-04-03，整理时间：2023-01-28

在LinuxMint中访问windows分区时报错，如下图：

![01.png](/img/computer/01-01.png)

表示/dev/sda5这个分区挂载失败，遇到这种情况，使用一条命令就可以解决了：

```shell
sudo ntfsfix /dev/sda5
```

操作结果如下图所示

![02.png](/img/computer/01-02.png)
