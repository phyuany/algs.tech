---
title: 装windows+Linux双系统后，Windows时间不正确的解决办法
date: 2023-01-31 00:33:00 +0800
categories: [技巧]
tags: [双系统]
pin: false
---

如标题所示，安装Linux+Windos双系统后，Windows时间会慢8个小时，在windows上使用管理员身份，在命令行终端执行以下命令即可

```bat
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```

以上指令是将windows识别硬件时间改以UTC-0为准
