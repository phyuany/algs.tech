---
title: 如何在Debian系统上安装PVE，实现与Windows的双系统共存
date: 2024-10-20 00:26:00 +0800
categories: [操作系统与网络]
tags: []
pin: false
---

## 一、需求概述与硬件配置

### 1.1 需求概述

我有一台电脑，安装了 `Windows` 和 `Debian` 双系统。`Windows` 系统主要用于日常娱乐，如打游戏、剪辑视频等操作，而 `Debian` 系统则主要用于开发、学习和工作，提供了一个高效的开源环境。

然而，为了进一步提升资源利用效率，并且方便同时处理更多任务，我希望在 `Debian` 系统上安装 `Proxmox Virtual Environment (PVE)`。`PVE` 作为一款企业级虚拟化平台，不仅支持 `KVM` 和 `LXC` 两种虚拟化技术，还提供了强大的 `Web` 界面管理，能让我轻松在同一台主机上创建和管理多个虚拟机。这不仅能在开发过程中模拟多种环境，还能大幅优化硬件资源的使用。

与常见的虚拟化软件如 `VirtualBox`、`VMware Workstation` 或 `Multipass` 之类的虚拟化工具相比，`PVE` 在资源管理、性能和灵活性方面有明显优势。`VirtualBox` 和 `VMware` 等桌面虚拟化软件更适合轻量级的虚拟化应用，但当需要运行多个虚拟机并管理资源时，它们的性能往往会受到限制。而 `PVE` 作为专门用于服务器虚拟化的平台，可以使用`Linux` 内核级别的虚拟化功能，不仅更高效，还能更好地利用 `Debian` 系统的强大功能。

在本文中，我将介绍如何将 `PVE` 安装在 `Debian` 系统上，以实现与 `Windows` 与 `Debian with PVE` 的双系统的共存。

### 2.1 硬件配置

| 类型 |                 型号                  | 备注 |
| ---- | ------------------------------------- | ---- |
| CPU  | Intel 12600k 12核16线程               |      |
| 内存 | 金百达银爵 16G DDR4 3200MHz * 4 总64G |      |
| 显卡 | NVIDIA 4060                           |      |
| 硬盘 | 致钛 7100pro 1T + 惠普某型号固态 4T   |      |

以上硬件只是作为参考，硬盘最终要分配给操作系统使用，这里特指硬盘，硬盘如何分配要根据自己电脑与自身需求的实际情况。

## 二、Debian系统安装

### 2.1 安装Debian

首先，我的电脑硬盘使用了 `GPT` 分区表格式，这是现代操作系统普遍推荐的格式化方式。与传统的 `MBR` 不同，`GPT` 格式支持更大的硬盘和更多的分区，并且需要通过 `UEFI` 引导系统。

在 `Windows` 系统中，通过官方安装程序创建的启动盘会自动进行分区，其中包含一个 `EFI` 分区（可以在磁盘管理工具中查看）。该分区存储了 `Windows` 的引导程序，是系统启动的关键。在安装 `Debian` 系统时，`Debian` 的引导程序也会被写入到这个 `EFI` 分区，从而实现双系统共存和引导管理。

我的 `Windows` 安装在 `1T` 的固态硬盘上，另外 `4T` 作为备用硬盘，或者说数据盘。这里建议选择性能好的硬盘用来安装系统，剩余的硬盘用来存储数据。我从 `Windows` 主分区中压缩了 `600G` 用于安装 `Debian` 系统。

### 2.2 Debian系统安装基本软件

`Debian` 的安装过程这里不再说明，`Debian` 的安装程序很方便，直接按照提示操作即可。我们可以选择安装桌面环境，不过如果你使用的是 `Nvidia` 的显卡，建议先安装系统基本组件，等进入系统后，安装显卡驱动之后再安装桌面环境。以下是我的安装过程：

首先安装好 `Debian` 之后推荐把 `apt` 源换为国内的源，这样速度会更快。并且安装是基本的软件，如果你的显卡是 `Nvidia`，那么需要额外安装 `Nvidia` 的驱动。建议使用 `non-free` 源直接安装即可，我的 `Nvidia` 显卡主机安装过程如下

```bash
# 软件源替换为中科大源
echo "deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware" > /etc/apt/sources.list
# 更新源
apt update
# 安装vim、网络工具、源代码编译基础依赖，以及nvidia显卡驱动
apt install -y vim net-tools build-essential nvidia-driver
# 安装gnome桌面，以下会安装桌面环境和常用桌面软件
apt install -y task-gnome-desktop
```

执行 `apt install -y task-gnome-desktop` 这条命令 `Debian` 会安装上常用的桌面环境，包括一些小游戏，如果不想要小游戏，可以执行以下命令删除掉

```bash
apt purge gnome-games
apt autoremove
```

如果你指向安装 `Debian` 的最基本桌面环境，不需要额外的工具，可以将上述最后一行安装命令替换如下

```bash
apt install -y gnome-core
```

执行完以上步骤，则完成基本软件以及 `Debian` 桌面环境的安装，重启后即可登录桌面环境。

## 三、PVE环境安装

你可以参考官网教程<https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm>

这里我根据官网教程以及个人需求，我的安装过程记录如下

### 3.1 设置本机域名解析

将本机 IP 地址解析为域名，方便后续使用，如下

```shell
echo "127.0.0.1 pve.local" >> /etc/hosts
```

域名需要根据你实际需求来设置，这里我设置为 `pve.local`。

### 3.2 安装PVE源

首先，新增 `PVE` 仓库 `key` 到系统中，如下

```shell
wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg 
```

然后添加 `PVE` 官网源地址到 `apt` 源列表中，如下

```shell
echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
```

如果在国内，更推荐使用镜像，我使用的是中科大源的源地址，如下

```shell
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```

### 3.3 更新源并安装PVE

更新源并安装 `PVE`，如下

```shell
# 更新软件源
apt update && apt full-upgrade
# 安装PVE
apt install proxmox-default-kernel proxmox-ve
# 删除多余的内核
apt purge linux-image-amd64
apt autoremove
# 更新引导程序
update-grub
```

`os-prober` 用于包扫描主机的所有分区，以创建双引导 `GRUB` 条目。但是扫描的分区也可以包括分配给虚拟机的分区，这些分区不想作为引导条目添加，所以官方推荐删除 `os-prober` 包，如下命令

```shell
apt purge os-prober
apt autoremove
```

正常情况下，完成以上操作虚拟机就可以正常使用了，通过访问 `https://127.0.0.1:8006` 地址即可打开 `PVE` 管理后台。

## 四、配置硬件直通

### 4.1 什么是硬件直通？

`PVE（Proxmox Virtual Environment）` 中的硬件直通 `（PCI Passthrough）` 指的是将主机的某个硬件设备（如显卡、网络适配器等）直接分配给虚拟机，使虚拟机能够直接控制该硬件。这种方式能够提高虚拟机的性能，因为虚拟机可以直接使用物理设备的能力，而不需要通过虚拟化层进行访问。常用于需要高性能计算的场景，如游戏、图形处理和高性能计算任务。实现硬件直通通常需要支持 `VT-d` 或 `AMD-Vi` 的处理器和主板，并在 `BIOS` 中启用相关设置。

我们需要在系统中启用 `IOMMU（Input-Output Memory Management Unit）`，这对于 `PCI` 设备直通是必需的。下文是硬件直通配置的步骤。

### 4.2 启用 BIOS 中的 IOMMU

- 重启计算机，进入 BIOS 设置。
- 查找与 `IOMMU` 相关的选项，通常在 “`Advanced`” 或 “`Chipset`” 设置中。
- 启用 `IOMMU`（可能标记为 `VT-d` 或 `AMD-Vi`，具体取决于你的 `CPU`）。
- 保存并退出 `BIOS` 设置。

### 4.3 修改 GRUB 配置

- 在 Debian 系统中，打开终端。
- 编辑 GRUB 配置文件：

```bash
sudo vim /etc/default/grub
```

- 找到 `GRUB_CMDLINE_LINUX_DEFAULT` 行，并添加以下内容（根据你的 CPU 类型选择）：

对于 Intel CPU：

```shell
intel_iommu=on
```

对于 AMD CPU：

```shell
amd_iommu=on
```

修改后的行可能类似于

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```

保存并退出编辑器，运行以下命令以更新 GRUB 配置

```bash
sudo update-grub
```

### 4.4 重启 PVE 系统

重启计算机，使更改生效。启动后，你可以通过以下命令检查 `IOMMU` 是否已启用

```bash
dmesg | grep -e DMAR -e IOMMU
```

如果启用，应该能看到相关的信息。完成上述步骤后，`IOMMU` 应该会正常启用，你可以尝试添加 `PCI` 直通设备。如果仍然遇到问题，请检查 `BIOS` 设置是否保存成功，并确保你的硬件支持 `IOMMU`。

## 五、无线网卡IP分配

### 5.1 网段分配

我的主机使用无线网连接的路由器，而无线网通常不支持虚拟网桥绑定。为了使得宿主机能通过网络正常访问虚拟机，且保证虚拟机能访问外网。我们通过使用创建一个不绑定网卡的虚拟网桥，使用 `NAT` 的方式实现网络地址转换，需要在 `Proxmox VE (PVE)` 宿主机上设置执行下文的步骤。

根据我的电脑硬件以及家庭网络环境的情况，我的宿主机和虚拟机的计划IP或网段分配如下

|    设备    |     IP或网段     |
| ---------- | ---------------- |
| 路由器     | 192.168.0.1      |
| 宿主机     | 192.169.0.100    |
| 虚拟机网桥 | 192.168.100.1    |
| 虚拟机     | 192.168.100.0/24 |

### 5.2 创建NAT网络

1. 登录到 Proxmox Web 界面。
2. 在左侧导航栏中，选择 **“数据中心”**。
3. 选择 **“网络”** 标签。
4. 点击 **“创建”** 按钮，选择 **“Linux Bridge”**。
5. 为网桥命名（如 `vmbr1`），确保 **“桥接端口”** 留空。
6. 点击 **“添加”**。

### 5.3 配置 NAT

1.打开 Proxmox 的终端或 SSH 登录到宿主机。
2.编辑 `/etc/network/interfaces` 文件，添加如下配置：

```bash
auto vmbr1
iface vmbr1 inet static
    address 192.168.100.1
    netmask 255.255.255.0
    bridge_ports none
```

3.保存文件并退出。

### 5.4 启用 IP 转发

1.运行以下命令启用 IP 转发：

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### 5.5 配置 iptables 规则

1.设置 iptables 规则以允许 NAT：

```bash
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o <你的外网接口> -j MASQUERADE
```

将 `<你的外网接口>` 替换为宿主机的实际外网接口（我的无线网口是 `wlo1`）。

宿主机重启后，iptables 规则被重置是因为这些规则并没有被保存。要确保在重启后保持 NAT 规则，可以按照以下步骤操作。

使用以下命令将当前的 iptables 规则保存到文件中：

```shell
iptables-save > /etc/iptables/rules.v4
```

如果没有 `/etc/iptables/rules.v4` 文件，系统可能需要安装 `iptables-persistent` 包，该包可以在系统启动时自动加载这些规则：

```shell
apt install iptables-persistent
```

在安装过程中，系统会询问您是否要保存当前的规则。选择“是”以确保当前规则在重启后仍然生效。

2.为了使规则在重启后生效，可以将这些规则写入 `/etc/rc.local` 或使用 `iptables-persistent`。

### 5.6 配置虚拟机

1. 在 PVE 中选择你的虚拟机，进入 **“硬件”** 选项。
2. 点击 **“添加”** > **“网络设备”**。
3. 将网络设备连接到刚创建的 `vmbr1`。
4. 启动虚拟机。

### 5.7 在虚拟机中配置网络

1. 在虚拟机中，设置一个静态IP（如 `192.168.100.2`），子网掩码为 `255.255.255.0`，网关为 `192.168.100.1`。

### 5.8 测试连接

在虚拟机中，尝试 ping 外部地址，如 `ping 192.168.0.1`以确认能访问内网路由器，`ping www.baidu.com`以确保能访问互联网。

## 六、修复Windows引导

### 6.1 修改grub配置

在安装 pve 之后，grub 中的 window 引导条目将丢失。即使系统安装了 `os-prober` 也没有扫描出来，我不知道是我主机的本身的原因，还是是因为我所安装 pve 版本的原因。实际上我们有办法手动添加windows引导。

一般 Windows 的 EFI 系统分区是第一个硬盘的第一个分区。你可以在 `/etc/grub.d/40_custom` 中使用以下配置来添加 Windows 引导：

```bash
menuentry 'Windows 10' {
    insmod part_gpt
    insmod fat
    set root='hd0,gpt1'  # hd0 指的是第一块磁盘，gpt1 表示 GPT 分区表中的第一个分区
    chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
```

之后执行：

```bash
update-grub
```

看看是否能看到 Windows 的引导选项。如果仍有问题，一般是你指定的硬盘位置不正确，以在 GRUB 命令行界面手动测试引导，

### 6.2 grub页面手动测试引导

在 GRUB 启动时，按下 c 进入命令行，手动尝试引导 Windows：

```bash
set root=(hd0,gpt1)
chainloader /EFI/Microsoft/Boot/bootmgfw.efi
boot
```

如果报错，可以尝试修改一下 `root` 参数，如`hd0` 修改 `hd1`，再尝试。
