---
title: 不使用激活工具，怎样使用代码激活Windows10？
date: 2023-01-31 00:28:00 +0800
categories: [技巧]
tags: []
pin: false
---

## 一、概述

现在我们买的笔记本电脑中，一般情况下默认出厂安装的是Windows 10家庭中文版。

有时候我们系统崩溃或者其他原因我们需要重装系统，如果我们重装的是Windows 10系统家庭中文版，联网之后系统会自动激活，因为我们的电脑主板已经默认绑定了一个正版的Windows 10家庭中文版。

但是如果我们安装器其他版本，比如：企业版、专业版，那么系统将处于“未激活”状态。本文将给大家介绍，如何使用命令来激活试用。

如果你需要在笔记本上安装正版Windows 10，可以参考这个视频教程：[https://www.bilibili.com/video/BV187411A7rt](https://www.bilibili.com/video/BV187411A7rt)

如果你的系统崩溃了，需要从崩溃的系统中备份出重要文件，可以参考这个教程：[https://www.bilibili.com/video/BV1D7411E7ZC](https://www.bilibili.com/video/BV1D7411E7ZC)

## 二、激活步骤

如果我们使用的是其他版本的Windows，可以手动激活试用

- 以管理员身份打开命令提示符

单击开始按钮，搜索`cmd`，然后以管理员权限运行它。
![01.png](/img/skill/03-01.png)

- 安装KMS客户端密钥

使用命令`slmgr /ipk 你的秘钥`安装许可证密钥（您的许可证密钥是与Windows版本相对应的激活密钥）。以下是Windows 10批量许可证密钥的列表。

```text
家庭： TX9XD-98N7V-6WMQ6-BX7FG-H8Q99
家庭N： 3KHY7-WNT83-DGQKR-F7HPR-844BM
家庭单一语言： 7HNRX-D7KGG-3K4RQ-4WPJ4-YTDFH
特定国家版： PVMJN-6DFY6-9CCP6-7BKTT-D3WVR
专业： W269N-WFGWX-YVC9B-4J6C9-T83GX
专业N： MH37W-N47XK-V7XM9-C7227-GCQG9
教育： NW6C2-QMPVW-D7KKK-3GKT6-VCFB2
教育N： 2WH4N-8QGBV-H22JP-CT43Q-MDWWJ
企业： NPPR9-FWDCX -D2C8J-H872K-2YT43
企业N： DPH2V-TTNVB-4X9Q3-TJR4H-KHJW4
```

注意：您需要按[Enter]键才能执行命令。

![02.png](/img/skill/03-02.png)

- 设置KMS机器地址

使用命令`slmgr /skms kms8.msguides.com`（或者`slmgr /skms s8.now.im`）连接到网上的KMS服务器

![03.png](/img/skill/03-03.png)

- 激活你的Windows

最后一步是使用命令`slmgr /ato`激活Windows。

![04.png](/img/skill/03-04.png)

## 三、总结

免责声明：本文只供学习参考，由于个人的不正当使用造成的后果与作者无关，建议购买正版系统。

> 参考链接：[https://msguides.com/microsoft-software-products/2-ways-activate-windows-10-free-without-software.html](https://msguides.com/microsoft-software-products/2-ways-activate-windows-10-free-without-software.html)
