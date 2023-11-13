---
title: 怎样安装ArchLinux以及Deepin桌面环境
date: 2023-01-28 00:29:00 +0800
categories: [操作系统与网络]
tags: [archlinux]
pin: false
---

> 撰写时间：2020-02-11，整理时间：2023-01-28

## 一、概述

Arch Linux  是一个轻量级的Linux发行版本，实际上，Arch Linux提供给用户很多选择，用户可以自定义自己的安装内容，不像其他很多的Linux发行版本，安装过程甚至是一个只有“下一步”的傻瓜式操作，因此我觉得Arch Linux是我见过安装过程最有技术含量的Linux发行版本。不过我们可以从中学到很多东西，因为很多东西是我们亲手构造出来的。Arch Linux的软件包管理工具是pacman，接下来我们基于Arch Linux镜像自带的Linux工具包以及pacman，从零构建属于自己的Arch Linux。

安装过程包括以下3个步骤：

- 进入镜像磁盘，连接网络

- 配置分区，安装基础环境

- 安装好之后进入本地系统，配置系统，安装基本工具

- 安装图形界面

## 二、准备工作

### 2.1 选择安装方式

选择1：在你原来的操作系统上安装一个虚拟机软件，用虚拟机安装

选择2：在你的实体电脑，使用全新的硬盘安装全新的Arch Linux单系统

选择2：在你的实体电脑，在原有windows操作系统的基础之上，压缩出一块空间，安装双系统

### 2.2 下载安装镜像

官方下载地址[https://www.archlinux.org/download/](https://www.archlinux.org/download/)
在写这篇文章的时候，官方最新安装包为  archlinux-2020.02.01-x86_64.iso

### 2.3 注意事项

- 安装Arch Linux的时候，应当连接到互联网，因为我们需要从网络上下载自定义内容，比如连接WIFI，或者连接有线网络
- 如果是安装在实体机，需要进入BIOS将安全启动（secure boot）关闭掉
- 启动方式为UEFI，如果你实体电脑没开启UEFI启动方式，需要进入BIOS设置一下；虚拟机则需要在虚拟机工具中设置

### 2.4 制作启动盘

如果你安装在电脑上的空硬盘或者基于windows安装双系统的话，简单的说就是不是装在虚拟机上，而是装在实体机上，那么就需要使用系统镜像制作成U盘启动器。有很多制作启动盘的工具，我这里推荐的是usbwriter这个工具,以下是下载地址：[https://sourceforge.net/projects/usbwriter/](https://sourceforge.net/projects/usbwriter/)

如果你安装在虚拟机之上，直接使用原镜像文件就可以了

## 三、安装详细过程

### 3.1 进入镜像磁盘，连接网络

启动镜像系统后，我们直接按回车进去。可以看到如下图所示：
![01.png](/img/computer/04-01.png)

Arch Linux安装镜像自带一些常用的工具集合，我们可以直接使用自带工具连接网络。如果使用虚拟机安装，那么你的电脑主机连接网络之后，Arch 安装镜像的系统自然会自动连接上网络。如果你使用真机安装，那你需要手动连接网络。

- 查看网络设备

```shell
ip link
```

通过这个命令显示你的电脑网络硬件设备
![02.png](/img/computer/04-02.png)

因为我在虚拟机的环境中，如果你在实体笔记本的安装环境中，应该有一个wlan0

- 打开wifi网络

```shell
ip link set wlan0 up
```

如果wifi启动不了，出现rfkill错误时，执行以下命令

```shell
rfkill unblock all
```

- 扫描附近WIFI列表

```shell
iwlist wlan0 scan
```

不过执行以上命令会出现很多次要信息，可以在执行扫面WIFI命令时可以设置ESSID（WIFI名称）过滤条件

```shell
iwlist wlan0 scan | grep ESSID
```

- 使用wpa_supplicant链接WIFI

首先使用wpa_passphrase命令生成wifi连接的配置文件

```shell
wpa_passphrass B17 wk12345 > internet.conf
```

以上命令中，B17代表WIFI名称，wk123456代表WIFI密码，然后使用使用对应的配置文件在后台连接WIFI

```shell
wpa_supplicant -c internet.conf -i wlan0 &
```

-c参数指定的是wifi配置文件，-i参数指定使用的设备，即 wlan0，&符号表示后台运行
网络连接成功之后，使用dhcpcd命令自动获取id地址

```shell
dhcpcd &
```

- 检测网络是否连接成功

使用ping命令检测即可

```shell
ping www.baidu.com
```

### 3.2 配置分区，安装基础环境

假设现在磁盘是空的，我们需要创建三个分区，分别是EFI分区（用于存放系统引导程序）、swap分区（用于给让系统支持虚拟内存）、主分区（用于安装系统）

- 查看磁盘情况

```shell
fdisk -l
```

可以看到如下图所示
![03.png](/img/computer/04-03.png)

我们可以看到/dev/sda就是我们的空磁盘，大小为16GiB，选择空磁盘，将对其进行操作

```shell
fdisk /dev/sda
```

系统会提示你执行相应的命令，如果你不知道具体的用法，可以使用输入“m”指令查看相关帮助，如下图所示

![04.png](/img/computer/04-04.png)

- 创建EFI

我们使用“n”指定创建一个新I分区，如下图

![05.png](/img/computer/04-05.png)

此时提示输入选择分区类型，“p”指令代表主分区，“e”指令代表扩展分区，默认为“p”指令，我们直接回车即可。回车后系统提示选择分区序号，我们选择默认序号即可，再按回车之后系统提示选择开始节点，我们直接按回车选择默认即可，如下图

![06.png](/img/computer/04-06.png)

可以看到此时系统提示选择分区的末尾节点，默认选的的是最后一个节点，所以我们需要手动输入，也可以使用加减（+/-）的方式进行选择，我们选择“+512M”，结果如图所示

![07.png](/img/computer/04-07.png)

此时系统已经保存我们的创建信息，不过没有输入“w”指令之前就退出是不生效的，接下来继续创建swap分区

- 创建swap分区

swap分区用来做虚拟内存分区，可以根据你系统的硬件来设置，不过一般情况下设置为实际内存的一般就可以了。我们继续执行以上的流程，只是选择分区大小的时候选择“+1G”就可以

- 创建系统主分区

创建分区的指令和以上的指令一致，只是在选择分区大小的时候你要注意，如果你以后还安装其他系统（如windows）你就需要预留一下一个空间，否则直接按回车即可，系统会选择剩下所有的空间。

- 保存分区

创建完分区之后，输入“w”指令进行保存，否则无效。最后我们使用fdisk -l 命令查看磁盘，可以看到，已经分好了分别为 /dev/sda1、/dev/sda2、/dev/sda3的三个分区了，如图所示

![08.png](/img/computer/04-08.png)

- 格式化分区

将/dev/sda1格式化为启动引导分区

```shell
mkfs.fat -F32 /dev/sda1
```

将/dev/sda2格式化为swap分区

```shell
mkswap /dev/sda2
```

开启swap分区

```shell
swapon /dev/sda2
```

将/dev/sda3格式化为Linux主分区

```shell
mkfs.ext4 /dev/sda3
```

- 安装系统

编辑软件库镜像源，我们将所有包含中国镜像的代码行剪切到文件顶部即可

```shell
vim /etc/pacman.d/mirrorlist
```

先将本地磁盘挂载到镜像系统中，将硬盘主分区挂载到镜像系统中

```shell
mount /dev/sda3 /mnt
```

将EFI分区挂载到镜像系统中

```shell
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

使用pacstrap执行安装操作

```shell
pacstrap /mnt base linux linux-firmware
```

在上面代码中，base linux linux-firmware是系统基础环境，base-devel是linux的工具集合，net-tools是网络工具包，安装完成以后，根据文件目录生成分区表文件

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

到此，基础环境已经安装到本地磁盘，下面我们需要进行相关配置

### 3.3 进入本地系统，安装基本工具

进入本地系统

```shell
arch-chroot /mnt
```

我们首先安装vim工具，我们将用vim工具来编辑系统的配置文件

```shell
pacman -S vim
```

- 更改时区

更改时区

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

同步时间

```shell
hwclock --systohc
```

- 配置本地化

首先编辑本地化字符集配置文件

```shell
vim /etc/locale.gen
```

找到下面这两行，并去掉“#”执行取消注释，如下

```shell
#en_US.UTF-8 UTF-8
```

```shell
#zh_CN.UTF-8 UTF-8
```

生成本地配置

```shell
locale-gen
```

- 先将系统语言设置为英文

```shell
vim /etc/locate.conf
```

添加以下内容

```shell
LANG=en_US.UTF-8
```

- 修改本地系统的名称，修改hosts文件

编辑本机名称

```shell
vim /etc/hostname
```

加入以下内容，我把我的arch系统叫做pan-PC

```shell
pan-PC
```

修改hosts文件

```shell
vim /etc/hosts
```

添加以下内容

```shell
127.0.0.1 localhost
127.0.0.1 pan-PC
::1       localhost
```

- 修改本地系统的root密码

使用passwd命令修改密码

```shell
passwd
```

- 安装bootloader

```shell
pacman -S grub efibootmgr amd-ucode os-prober
```

其中，grub是ArchLinux启动引导程序，即开机之后看到的那个界面，efitbootmgr是efi启动管理工具，amd-ucode用于厂家提供处理器级别的驱动更新（如果是intel处理器要安装intel-ucode），os-prober用来寻找电脑中的其他引导程序。接下来创建引导配置文件

```shell
mkdir /boot/grub
grub-mkconfig > /boot/grub/grub.cfg
```

安装grub，先查看系统类型

```shell
uname -m
```

可以看到显示

```shell
x86_64
```

于是我们要安装对应的efi

```shell
grub-install --target=x86_64-efi --efi-directory=/boot
```

- 安装互联网工具包

```shell
pacman -S wireless_tools wpa_supplicant dhcpcd
```

- 安装常用的开发工具集合

```shell
pacman -S base-devel
```

此时已经完成安装，使用exit指令推出本地系统，完成安装。关机，去除镜像系统，再次启动即可从grub引导进入ArchLinux，如下图
![09.png](/img/computer/04-09.png)

- 后续操作

一般情况下，个人电脑应当设置一个权限比较低的用户账号，我设置用用户名为"pan"，并为pan设置密码

```shell
useradd -m -g wheel pan
passwd pan
```

### 3.4 安装图形界面

- 安装中文字体（文泉驿）

安装桌面环境后你应该会用到切换成中文系统，因此我们安装一下中文字体

```shell
pacman -S wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy-zenhei
```

- 安装xorg

简单的说，有xorg才有桌面环境

```shell
pacman -S xorg
```

- 安装deepin桌面以及基本程序

```shell
pacman -S deepin deepin-extra file-roller firefox
```

deepin-extra是deepin自带的应用程序，如深度终端，file-roller是deepin自带的文件解压工具，不然使用右键提取文件时不管用，firefox是火狐浏览器

- 安装中文输入法

先安装fcitx

```shell
pacman -S fcitx fcitx-im fcitx-configtool
```

安装google拼音输入法，google拼音输入法依赖fictx

```shell
pacman -S fcitx-googlepinyin
```

配置输入法相关的环境变量

```shell
vim /etcc/rpofile
```

在文件末尾添加以下内容

```shell
export XIM=fcitx
export XIM_PROGRAM=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

- 设置自启登录界面

```shell
vim /etc/lightdm/lightdm.conf
```

找到greeter-session取消注释，并设置值

```shell
greeter-session = lightdm-deepin-greeter
```

设置lightdm开机自启

```shell
systemctl enable lightdm
```

至此，安装结束，重启之后就自动打开Deepin登录界面了。
