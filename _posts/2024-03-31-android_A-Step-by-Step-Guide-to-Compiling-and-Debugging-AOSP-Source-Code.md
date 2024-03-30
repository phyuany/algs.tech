---
title: 手把手教你编译和调试AOSP源码
date: 2024-03-31 04:12:00 +0800
categories: [Android]
tags: []
pin: false
---

## 一、下载AOSP源码

如果你电脑没有repo工具需要先安装 `repo` 工具，命令如下

```shell
mkdir ~/bin
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo
chmod +x ~/bin/repo
# 以下这条命令可以添加到 `~/.bashrc` 文件中
export PATH=~/bin:$PATH
```

假设电脑已经包含了git工具，接下来需要同步源代码，命令如下

```shell
mkdir aosp 
cd asop
# 初始化repo，指定源码分支为 `android-10.0.0_r41`
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r41
# 同步远程代码
repo sync
```

同步源码的耗时较长，耐心等待即可。源码同步完成后，即可进行第二步以后的操作。

## 二、编译SDK

初始化 `AOSP` 编译环境与选择 `SDK` 作为编译目标

```shell
# 初始化环境
source build/envsetup.sh
# 选择编译目标
lunch sdk-eng
```

终端输出信息如下

```shell
pan@pan-PC:~/Android/aosp/android13$ lunch sdk-eng

============================================
PLATFORM_VERSION_CODENAME=VanillaIceCream
PLATFORM_VERSION=VanillaIceCream
TARGET_PRODUCT=sdk
TARGET_BUILD_VARIANT=eng
TARGET_ARCH=x86
TARGET_ARCH_VARIANT=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.15.77-amd64-desktop-x86_64-Deepin-20.9
HOST_CROSS_OS=windows
BUILD_ID=AOSP.MASTER
OUT_DIR=out
===========================================
```

编译sdk

```shell
make sdk
```

最后生成的sdk目录在 `out/host/linux-x86/sdk/sdk/android-sdk_eng.{$USER}_linux-x86` 下，用于 `AOSP` 的后续开发和调试。

## 三、编译AOSP并启动模拟器

初始化 AOSP 编译的目标运行设备，这里我们选择 `aosp_x86_64-eng`

```shell
lunch aosp_x86_64-eng
```

编译

```shell
# 使用 make 数值 的命令进行指定线程数编译，也可以使用 m 命令自动选择最大线程数
make -j$(nproc)
```

编译完成之后，使用如下命令启动模拟器

```shell
emulator
```

## 四、使用ASFP打开AOSP的子模块

我们先介绍 `ASFP`，它是 `Android Studio for Platform` 的简称，谷歌官方提供了该工具，可以方便的进行基于 `AOSP` 的开发。如果你电脑上还没有安装 `ASFP`，可以到 [https://developer.android.google.cn/studio/platform](https://developer.android.google.cn/studio/platform) 下载安装。

这里我们要打开 `Setting` （系统设置APP）源代码并进行调试，步骤如下：

- 首页点击：Import Asfp Project

- 填写项目信息

项目关键信息如下图红标部分

![20240331031654](/img/android/20240331031654.png)

配置完成之后，点击 `Finish` 打开到代码编辑器窗口。

- 配置SDK

（1）打开 `Project Struct` -> `Platform Settings` -> `SDKs` -> `+` -> `Add Android SDK`：添加 `Android SDK`，选择 `out/host/linux-x86/sdk/sdk/android-sdk_eng.{$USER}_linux-x86`，添加后可重命名为 `aosp10-sdk`，点击 `Apply`。

（2）打开左侧的Project，配置SDK，选择刚才添加的SDK。

（3）`Project Settings` -> `Modules`，确保 `Settings` 模块已经使用了上述配置的SDK。这个时候项目就配置好了。

## 五、调试代码

回到代码主编辑器窗口后，首先找到关键的代码打断点，然后点击 `Attach Debugger to Android Process` 按钮连接到模拟器，就可以看到代码的调试效果了。如下图：

![20240331033429](/img/android/20240331033429.jpg)

在上图中，我打断点的位置是`Settings` 模块的 `SettingsHomepageActivity` 类的 `onCreate` 方法。该方法在启动系统设置时被调用，用来初始化界面。

点击 `OK` 后，我们需要在虚拟机上手动打开系统设置，然后就可以进行调试了。效果如下图：

![20240331034151](/img/android/20240331034151.png)
