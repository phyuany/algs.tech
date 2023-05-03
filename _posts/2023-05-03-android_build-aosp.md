---
title: AOSP编译流程
date: 2023-05-03 12:13:00 +0800
categories: [Android]
tags: [aosp]
pin: false
---

## 一、概述

在撰写这篇文章的时候，AOSP的最新版本是Android 13，本文将以Android 13为例，介绍AOSP的编译流程。理论上，电脑配置越高，编译速度越快，同时需要足够的磁盘空间。这里建议空余空间至少300G，否则编译过程中可能会出现磁盘空间不足的情况。操作系统建议使用`Ubuntu`或者国内的`Deepin`，以下是我电脑的配置：

- CPU：`12th Gen Intel(R) Core(TM) i5-12600K`
- 内存: `金百达银爵3200 16G * 2`
- 操作系统：`Deepin 20.9`

## 二、下载AOSP源代码

### 1.1 安装git

git主要用于下载AOSP源码，安装命令如下：

```shell
sudo apt install git
git config --global user.email i@phy.xyz
git config --global user.name phy
```

### 1.2 安装repo

repo 是 Google 开发的一个用于管理 Git 仓库的命令行工具。它使用多个 Git 仓库来管理大型项目的源代码，例如 Android Open Source Project (AOSP)。repo 工具不仅可以用于同步本地代码到仓库服务器，还可以将多个仓库代码合并成一个代码库，并进行代码控制和同步。以下是安装过程

```shell
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
```

`REPO_URL` 环境变量是用于指定 repo 工具默认使用的 AOSP 代码库的 URL 地址。如果不设置这个环境变量，repo 工具默认会从 Google 的源代码仓库中下载代码。而设置了这个环境变量，repo 工具会从指定的 URL 地址中下载代码。如设置清华的AOSP镜像地址，如下

```shell
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

### 1.3 下载源代码

#### 1.3.1 下载 AOSP 的月度版本

由于在国内，我们可以使用清华大学AOSP镜像站下载，我们直接下载AOSP最新备份的源代码，下载过程如下

```shell
curl -OC - https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录，这时目录里 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
repo sync # 正常同步一遍即可得到完整目录，或使用 repo sync -l 仅checkout代码
```

这一段代码是 AOSP 的下载和同步步骤。解释如下：

- `curl -OC - https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar`：使用 curl 命令从清华镜像站下载 AOSP 的月度版本，并保存到本地文件系统。
- `tar xf aosp-latest.tar`：使用 tar 命令解压 AOSP 的压缩文件，解压后会生成一个名为 AOSP 的目录。
- `cd AOSP`：进入 AOSP 目录。
- `repo sync`：使用 repo 工具同步 AOSP 源代码。repo sync 命令会从 AOSP 代码库中下载所有需要同步的代码，并自动进行合并和冲突解决。这是同步 AOSP 代码的推荐方式。

如果最后一步执行的是 `repo sync -l`，则会只进行本地同步，也就是只下载仓库中已经存在的代码，并不会从服务器上下载新增或者更改的代码。而如果执行的是 `repo sync`，则会从服务器上下载所有新增和更改的代码，并将其与本地仓库进行合并。这是同步最新的 AOSP 代码的常用方式，但需要注意，如果本地代码库已经很久没有同步过并且需要同步的代码比较多时，这个命令可能需要很长时间来完成同步操作。

#### 1.3.2 下载AOSP的指定版本

AOSP的版本列表：<https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds>。

下载某个特定的 Android 版本，如下

```shell
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-13.0.0_r31
```

同步源码树

```shell
repo sync
```

## 三、编译AOSP

### 3.1 安装编译依赖

AOSP的android 13版本需要使用python3，所以需要安装python3，如下命令

```shell
sudo rm /usr/bin/python /usr/bin/python2
sudo ln -s /usr/bin/python3.7 /usr/bin/python
sudo ln -s /usr/bin/python2.7 /usr/bin/python2
```

同时需要安装一些编译依赖，如下

```shell
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig libncurses5
```

我们需要安装一个JDK，后续可能需要使用到JDK工具，如下命令

```shell
sudo apt install openjdk-8-jdk
```

### 3.2 编译

```shell
source build/envsetup.sh
lunch sdk_phone_x86_64-userdebug
make -j14
export DISABLE_ARTIFACT_PATH_REQUIREMENTS="true"
emulator
```

这是一组用于编译和启动 AOSP SDK Phone x86_64 用户调试版本的代码和命令。按照顺序执行以下步骤：

1. 运行 `source build/envsetup.sh` 命令，该命令会将必要的环境变量和函数添加到当前 shell 中，使得后续的 AOSP 编译命令可以正常运行。

2. 运行 `lunch sdk_phone_x86_64-userdebug` 命令，该命令会设置编译目标为 SDK Phone x86_64 用户调试版本，并且会读取 build 目录下的 product 相关配置文件，以便后续的 `make` 命令可以正确编译 AOSP 系统。

3. 运行 `make -j14` 命令，该命令会启动 AOSP 编译过程，使用 14 个线程进行编译加速。这个过程可能需要很长时间，具体时间取决于计算机的配置和编译选项。

4. 运行 `export DISABLE_ARTIFACT_PATH_REQUIREMENTS="true"` 命令，该命令会禁用 AOSP 编译系统的一些路径要求检查，以便在后续启动模拟器时可以跳过一些限制。

5. 运行 `emulator` 命令，该命令会启动 AOSP 模拟器，并加载编译生成的系统镜像。在模拟器中可以进行系统测试、应用开发和调试等操作。

## 四、使用IDE导入源代码

### 4.1 概述

`mmm development/tools/idegen` 命令和 `development/tools/idegen/idegen.sh` 命令都是用于生成 IDE 配置文件的 AOSP 开发工具命令，但它们的使用场景和生成的配置文件类型略有不同。

具体来说，`mmm development/tools/idegen` 命令用于编译并生成 idegen 工具，这是一个用于生成 Eclipse 和 Android Studio 配置文件的工具。`mmm` 命令是指令 AOSP 编译系统编译特定的模块，而 `development/tools/idegen` 目录下的模块就是我们需要生成的 idegen 工具。该命令生成的是 `idegen.jar` 可执行文件，通过执行 `java -jar idegen.jar` 命令使用它。

而 `development/tools/idegen/idegen.sh` 命令则是 idegen 工具的一个封装脚本，用于生成特定 IDE 的配置文件。通过执行 `development/tools/idegen/idegen.sh` 命令，可以传递不同的参数来指定生成 Eclipse 或 Android Studio 的配置文件，例如：

```shell
development/tools/idegen/idegen.sh
development/tools/idegen/idegen.sh --ide=eclipse
development/tools/idegen/idegen.sh --android-studio
development/tools/idegen/idegen.sh --gradle
```

其中，第一个命令会默认生成适用于当前环境的 IDE 配置文件，第二个命令会生成 Eclipse 的配置文件，第三个命令会生成适用于 Android Studio 的配置文件，第四个命令则会生成适用于使用 Gradle 进行编译的工程的配置文件。

因此，`mmm development/tools/idegen` 命令适用于编译并生成 idegen 工具，而 `development/tools/idegen/idegen.sh` 命令则适用于生成特定 IDE 的配置文件。

### 4.2 生成IDE项目文件

```shell
mmm development/tools/idegen
```

会生成文件: `out/host/linux-x86/framework/idegen.jar`

### 4.3 生成IDEA配置文件

```shell
development/tools/idegen/idegen.sh
```

`development/tools/idegen/idegen.sh` 是真正的生成 IDE 项目文件的脚本，会在当前目录生成`android.iml`和`android.ipr`文件。

### 4.4 在AndroidStudio中打开

在AndroidStudio的open菜单打开到源码目录的`android.ipr`文件，系统将把项目加载到AS中。其实我们不进行源码编译，也能将源码导入AS中。导入后AS将会索引源代码，需要花费一段比较长的时间。
