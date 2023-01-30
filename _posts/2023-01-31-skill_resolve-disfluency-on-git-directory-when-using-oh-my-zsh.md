---
title: oh-my-zsh在git目录变得很卡的解决办法
date: 2023-01-31 00:48:00 +0800
categories: [技巧]
tags: [oh-my-zsh]
pin: false
---

在 oh-my-zsh 进入比较大的git目录时，会变得卡顿，原因是oh-my-zsh 要获取 git 更新信息

可以在git目录执行以下操作

- 设置 oh-my-zsh 不读取文件变化信息

```shell
git config --add oh-my-zsh.hide-dirty 1
```

- 设置 oh-my-zsh 不读取任何 git 信息

```shell
git config --add oh-my-zsh.hide-status 1
```
