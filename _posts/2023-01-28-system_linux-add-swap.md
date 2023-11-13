---
title: 怎样给Linux服务器添加交换空间
date: 2023-01-28 00:28:00 +0800
categories: [操作系统与网络]
tags: [swap]
pin: false
---

> 撰写时间：2019-11-12，整理时间：2023-01-28

演示系统：`Debian Server 9.9`，所有操作在root用户模式下

计算机中Swap空间也就是交换空间，Swap空间是电脑硬盘中的一部分，当计算机的实际内存不够用的时候，操作系统会去使用Swap空间，不过一般情况下Swap空间是用不着的。因为是硬盘上的一部分，所以Swap空间很慢。

## 一、检查时候有Swap空间

![深度截图_选择区域_20191112094648.png](/img/computer/03-01.png)

我们可以看到Swap空间为空

## 二、创建swap分区

创建2G的swap，可以根据你的服务器配置来调整大小，一般情况下，Swap空间不需要很大

```shell
dd if=/dev/zero of=/mnt/swap bs=1M count=2048  
```

设置交换分区文件

```shell
mkswap /mnt/swap
```

启动swap

```shell
swapon /mnt/swap
```

设置开机时自启用 swap 分区，需要修改文件 /etc/fstab 中的 swap 行，添加以下代码

```shell
/mnt/swap swap swap defaults 0 0
```

如图所示
![深度截图_20191112100328.png](/img/computer/03-02.png)

重启服务器之后，可以看到多出了swap空间
![深度截图_20191112100550.png](/img/computer/03-03.png)
