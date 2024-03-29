---
title: 如何为安卓设备编译 LineageOS 操作系统
date: 2024-03-29 22:59:00 +0800
categories: [Android]
tags: []
pin: false
---

## 一、概述

`LineageOS` 是一个基于 `Android` 的开源操作系统，支持各种设备。本文教程中使用的手机型号为 `Nubia Play 5G` 。可以通过 `LineageOS` 官网查看 `LineageOS` 官方支持的手机型号。同时 `LineageOS` 官网也提供了编译教程，这里就不再赘述。我们可以打开 [LineageOS](https://lineageos.org/) 官网查看更多信息。整个编译过程大概包含以下几个步骤：

- 准备编译环境
- 拉取源码
- 配置编译环境
- 执行编译
- 刷机

在撰写文章时，`LineageOS` 提供了 `LineageOS 20` 和 `LineageOS 21`两个稳定版本，这里我们选择 `LineageOS 20` 。下文详细介绍 `LineageOS 20` 的编译过程。

## 二、准备环境

### 1.1 电脑硬件与操作系统环境

我的电脑硬件配置如下：

| 硬件 |         配置         |                         备注                          |
| ---- | -------------------- | ----------------------------------------------------- |
| CPU  | intel 12600K         | 理论上CPU越好，编译效率越高。                         |
| 内存 | 金百达 3200 16GB * 2 | 建议32GB以上内存                                      |
| 硬盘 | 固态硬盘 4TB         | 建议1TB以上硬盘，编译过程会产生挺多文件，需要硬盘空间 |

操作系统：`Deepin 20.9`，官方推荐使用 `Ubuntu 20.04` ，使用相近的 `Linux` 版本并且能解决依赖问题就可以。

### 1.2 安装platform-tools

首先我们需要安装 `platform-tools`，它包含了我们刷机的使用到的基本工具，例如 `adb`、`fastboot` 。后续在刷机过程中，需要使用到 `adb`、`fastboot` 。

下载地址：[platform-tools](https://developer.android.google.cn/tools/releases/platform-tools)

### 1.3 安装依赖

在编译的过程中，需要使用到基础的环境依赖，安装命令如下：

```shell
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick lib32readline-dev lib32z1-dev libelf-dev liblz4-tool libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```

### 1.4 安装git

如果你电脑还没有git，使用如下命令安装

```shell
sudo apt install git
```

并且执行git初始化

```shell
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

### 1.5 安装repo

`repo` 是 `Android` 源码的版本控制工具，它类似于 `Git` ，但是更适合 `Android` 源码。我们通过国内的 `tuna` 镜像源下载，速度比较快。

```shell
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
chmod +x repo
```

为了方便可以将其拷贝到PATH的任意目录里。`repo` 的运行过程中会尝试访问官方的 `git` 源，如果想使用 `tuna` 的镜像源，可以将如下内容复制到你的 `~/.bashrc` 里并重启终端。

```shell
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

## 三、下载源代码

在一个空的目录下执行 `repo` 初始化

```shell
repo init -u https://github.com/LineageOS/android.git -b lineage-20.0 --git-lfs
```

为了提高拉取源代码的速度，需要执行以下操作：

打开.repo/manifests/default.xml，将

```xml
  <remote  name="github"
           fetch=".."
           review="review.lineageos.org" />
```

改成

```xml
  <remote  name="github"
           fetch="https://github.com/" />

  <remote  name="lineage"
           fetch="https://mirrors.tuna.tsinghua.edu.cn/git/lineageOS/"
           review="review.lineageos.org" />
```

将

```xml
  <remote  name="aosp"
           fetch="https://android.googlesource.com"
```

改成

```xml
  <remote  name="aosp"
           fetch="https://mirrors.tuna.tsinghua.edu.cn/git/AOSP"
```

将

```xml
  <default revision="..."
           remote="github"
```

改成

```xml
  <default revision="..."
           remote="lineage"
```

同步源码树（以后只需执行这条命令来同步）：

```shell
repo sync
```

拉取厂商依赖代码

```shell
# 创建厂商目录
mkdir -p vendor/nubia/ && cd vendor/nubia
# 拉取厂商代码
git clone -b lineage-20 https://github.com/nubia-development/proprietary_vendor_nubia_sm7250-common.git sm7250-common
# 回到源码目录
cd -
```

## 四、编译

初始化环境

```shell
# 初始化环境
source build/envsetup.sh
# 进行设备初始化，要联网，否则报错
breakfast nx651j
```

执行设备初始化命令过程输出如下

```shell
pan@pan-PC:~/Android/aosp/lineage$ breakfast nx651j
In file included from build/make/core/config.mk:353:
In file included from build/make/core/envsetup.mk:352:
build/make/core/product_config.mk:228: error: Can not locate config makefile for product "lineage_nx651j".
23:19:48 dumpvars failed with: exit status 1
Device nx651j not found. Attempting to retrieve device repository from LineageOS Github (http://github.com/LineageOS).
Found repository: android_device_nubia_nx651j
Default revision: lineage-20.0
Checking branch info
Using fallback branch: lineage-20
Checking if device/nubia/nx651j is fetched from android_device_nubia_nx651j
Adding dependency: LineageOS/android_device_nubia_nx651j -> device/nubia/nx651j
Syncing repository to retrieve project.
Fetching: 100% (1/1), done in 1.687s
repo sync has finished successfully.
Repository synced!
Looking for dependencies in device/nubia/nx651j
Default revision: lineage-20.0
Checking branch info
Using fallback branch: lineage-20
Adding dependencies to manifest
Checking if device/nubia/sm7250-common is fetched from android_device_nubia_sm7250-common
Adding dependency: LineageOS/android_device_nubia_sm7250-common -> device/nubia/sm7250-common
Syncing dependencies
Fetching: 100% (2/2), done in 5.317s
repo sync has finished successfully.
Looking for dependencies in device/nubia/sm7250-common
Default revision: lineage-20.0
Checking branch info
Using fallback branch: lineage-20
Default revision: lineage-20.0
Checking branch info
Using fallback branch: lineage-20
Adding dependencies to manifest
Checking if hardware/nubia is fetched from android_hardware_nubia
Adding dependency: LineageOS/android_hardware_nubia -> hardware/nubia
Checking if kernel/nubia/sm7250 is fetched from android_kernel_nubia_sm7250
Adding dependency: LineageOS/android_kernel_nubia_sm7250 -> kernel/nubia/sm7250
Syncing dependencies
Fetching: 100% (4/4), done in 4m14.499s
正在检出文件: 100% (71804/71804), 完成.
Checking out: 100% (4/4), done in 5.215s
repo sync has finished successfully.
Looking for dependencies in hardware/nubia
hardware/nubia has no additional dependencies.
Looking for dependencies in kernel/nubia/sm7250
kernel/nubia/sm7250 has no additional dependencies.
Looking for dependencies in hardware/samsung/nfc
hardware/samsung/nfc has no additional dependencies.
Done

============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=13
LINEAGE_VERSION=20.0-20240227-UNOFFICIAL-nx651j
TARGET_PRODUCT=lineage_nx651j
TARGET_BUILD_VARIANT=userdebug
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-2a
TARGET_CPU_VARIANT=generic
TARGET_2ND_ARCH=arm
TARGET_2ND_ARCH_VARIANT=armv8-a
TARGET_2ND_CPU_VARIANT=generic
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.15.77-amd64-desktop-x86_64-Deepin-20.9
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=TQ3A.230901.001
OUT_DIR=out
PRODUCT_SOONG_NAMESPACES=vendor/nubia/sm7250-common device/nubia/sm7250-common hardware/nubia vendor/qcom/opensource/usb/etc hardware/qcom-caf/sm8250 vendor/qcom/opensource/commonsys/display vendor/qcom/opensource/commonsys-intf/display vendor/qcom/opensource/display vendor/qcom/opensource/data-ipa-cfg-mgr-legacy-um vendor/qcom/opensource/dataservices
============================================
```

设置编译缓存

```shell
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
ccache -o compression=true
```

最后执行命令编译

```shell
# 回到源码根目录
croot
# 编译
brunch nx651j
```

## 五、总结

以上是源码编译的步骤，编译成功后，在`out/target/product/nx651j/`目录下就可以看到编译好的系统镜像，如果编译失败，请根据提示进行修改。编译完成后，可以使用`fastboot`、`adb` 等工具将镜像烧录到手机中。这里仅介绍编译步骤，更多编译相关知识，请自行搜索。
