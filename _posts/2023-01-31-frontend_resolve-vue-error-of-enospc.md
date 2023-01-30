---
title: "Error: ENOSPC: System limit for number of file watchers reached，Vue项目运行出现该报错的解决办法"
date: 2023-01-31 00:23:00 +0800
categories: [前端]
tags: []
pin: false
---

在Linux系统上运行vue项目。出现如题报错代码的解决办法，在终端执行以下命令

```shell
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
sudo sysctl --system
```
