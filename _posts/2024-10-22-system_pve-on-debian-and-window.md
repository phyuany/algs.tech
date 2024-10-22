---
title: 如何在Debian系统上安装PVE，实现与Windows的双系统共存
date: 2024-10-20 00:26:00 +0800
categories: [操作系统与网络]
tags: []
pin: false
---

## 一、需求概述

我有一台电脑，安装了 Windows 和 Debian 双系统。Windows 系统主要用于日常娱乐，如打游戏、剪辑视频等操作，而 Debian 系统则主要用于开发、学习和工作，提供了一个高效的开源环境。

然而，为了进一步提升资源利用效率，并且方便同时处理更多任务，我希望在 Debian 系统上安装 Proxmox Virtual Environment (PVE)。PVE 作为一款企业级虚拟化平台，不仅支持 KVM 和 LXC 两种虚拟化技术，还提供了强大的 Web 界面管理，能让我轻松在同一台主机上创建和管理多个虚拟机。这不仅能在开发过程中模拟多种环境，还能大幅优化硬件资源的使用。

与常见的虚拟化软件如 VirtualBox、VMware Workstation 或 Multipass 相比，PVE 在资源管理、性能和灵活性方面有明显优势。VirtualBox 和 VMware 等桌面虚拟化软件更适合轻量级的虚拟化应用，但当需要运行多个虚拟机并管理资源时，它们的性能往往会受到限制。而 PVE 作为专门用于服务器虚拟化的平台，不仅更高效，还能更好地利用 Debian 系统的强大功能。

在本文中，我将介绍如何将 PVE 安装在 Debian 系统上，以实现与 Windows 双系统的共存。

## 二、硬件

| 类型 |                 型号                  | 备注 |
| ---- | ------------------------------------- | ---- |
| CPU  | Intel 12600k 12核16线程               |      |
| 内存 | 金百达银爵 16G DDR4 3200MHz * 4 总64G |      |
| 显卡 | NVIDIA 4060                           |      |
| 硬盘 | 致钛 7100pro 1T + 惠普某型号固态 4T   |      |

以上硬件只是作为参考，硬盘最终要分配给操作系统使用，这里特指硬盘，硬盘如何分配要根据自己电脑的实际情况。

## 三、软件安装过程

### 3.1.安装debian

首先，我的电脑硬盘使用了 GPT 分区表格式，这是现代操作系统普遍推荐的格式化方式。与传统的 MBR 不同，GPT 格式支持更大的硬盘和更多的分区，并且需要通过 UEFI 引导系统。

在 Windows 系统中，通过官方安装程序创建的启动盘会自动进行分区，其中包含一个 EFI 分区（可以在磁盘管理工具中查看）。该分区存储了 Windows 的引导程序，是系统启动的关键。接下来，在安装 Debian 系统时，Debian 的引导程序也会被写入到这个 EFI 分区，从而实现双系统共存和引导管理。

我的 windows 安装在 1T 的固态硬盘上，另外 4T 作为备用硬盘，或者说数据盘。这里建议选择性能好的硬盘用来安装系统，剩余的硬盘用来存储数据。我从 windows 主分区中压缩了 600G 用于安装 Debian 系统。

### 3.2.在debian上安装pve

debian 的安装过程这里不再说明，debian 的安装程序很方便，直接按照提示操作即可。安装过程中，可以选择安装桌面环境。不过如果你使用的是 nvidia 的显卡，我建议先安装系统基本组件，等进入系统后，安装显卡驱动之后再安装桌面环境。以下是我的安装过程：

首先安装好 debian 之后推荐把 apt 源换为国内的源，这样速度会更快。并且安装是基本的软件，如果你的显卡是nvidia，那么需要额外安装 nvidia 的驱动。建议使用 `non-free` 源直接安装即可，

```powershell
# 软件源替换为中科大源
echo "deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware" > /etc/apt/sources.list
# 更新源
apt update
# 安装vim、网络工具、源代码编译基础依赖，以及nvidia显卡驱动
apt install -y vim net-tools build-essential nvidia-driver
# 安装gnome桌面，以下会安装桌面环境和常用桌面软件
apt install -y task-gnome-desktop
```

执行 `apt install -y task-gnome-desktop` 这条命令时，debian 会安装上常用的桌面环境，包括一些小游戏，如果不想要小游戏，可以执行以下命令删除掉

```shell
apt purge gnome-games
apt autoremove
```

在安装完 debian 之后，我们开始安装 PVE。

### 3.3.安装pve

你可以参考官网教程<https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm>

这里我再根据官网教程做个总结，安装过程如下

#### (3.3.1) 设置本机域名解析

在本机上，将本机 IP 地址解析为域名，方便后续操作。

```shell
echo "127.0.0.1 pve.local" >> /etc/hosts
```

域名需要根据你实际需求来设置，这里我设置为 pve.local。

#### (3.3.2) 安装pve源

首先，新增 pve 仓库 key 到系统，如下

```shell
wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg 
```

然后添加 pve 官网源地址到 apt 源列表中，如下

```shell
echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
```

如果在国内，更推荐使用镜像，我使用的是中科大源的源地址，如下

```shell
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```

#### (3.3.3) 更新源并安装pve

更新源并安装 pve，如下

```shell
# 更新软件源
apt update && apt full-upgrade
# 安装pve
apt install proxmox-default-kernel proxmox-ve
# 删除多余的内核
apt purge linux-image-amd64
apt autoremove
# 更新引导程序
update-grub
```

`os-prober` 包扫描主机的所有分区，以创建双引导 `GRUB` 条目。但是扫描的分区也可以包括分配给虚拟机的分区，这些分区不想作为引导条目添加。如果你没有安装 Proxmox VE 作为另一个操作系统的双引导，你可以删除 `os-prober` 包。如下命令

```shell
apt purge os-prober
apt autoremove
```

正常情况下，完成以上操作虚拟机就可以正常使用了，通过访问 `https://127.0.0.1:8006` 地址即可打开 PVE 管理后台。虚拟机的创建、组网、挂载虚拟硬盘、直通硬件等，可以在后台是实现。

## 四、组网情况

### 4.1. 网段分配

我的主机使用无线网连接的路由器，而无线网通常不支持虚拟网桥绑定，我的宿主机和虚拟机的计划IP或者网段分配如下

|    设备    |     IP或网段     |
| ---------- | ---------------- |
| 路由器     | 192.168.0.1      |
| 宿主机     | 192.169.0.100    |
| 虚拟机网桥 | 192.168.100.1    |
| 虚拟机     | 192.168.100.0/24 |

在 Proxmox VE (PVE) 上设置 NAT 的步骤如下。

### 4.2. 创建 NAT 网络

1. 登录到 Proxmox Web 界面。
2. 在左侧导航栏中，选择 **“数据中心”**。
3. 选择 **“网络”** 标签。
4. 点击 **“创建”** 按钮，选择 **“Linux Bridge”**。
5. 为网桥命名（如 `vmbr1`），确保 **“桥接端口”** 留空。
6. 点击 **“添加”**。

### 4.3. 配置 NAT

1. 打开 Proxmox 的终端或 SSH 登录到宿主机。
2. 编辑 `/etc/network/interfaces` 文件，添加如下配置：

```bash
auto vmbr1
iface vmbr1 inet static
    address 192.168.100.1
    netmask 255.255.255.0
    bridge_ports none
```

3. 保存文件并退出。

### 4.4. 启用 IP 转发

1. 运行以下命令启用 IP 转发：

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### 4.5. 配置 iptables 规则

1. 设置 iptables 规则以允许 NAT：

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

2. 为了使规则在重启后生效，可以将这些规则写入 `/etc/rc.local` 或使用 `iptables-persistent`。

### 4.6. 配置虚拟机

1. 在 PVE 中选择你的虚拟机，进入 **“硬件”** 选项。
2. 点击 **“添加”** > **“网络设备”**。
3. 将网络设备连接到刚创建的 `vmbr1`。
4. 启动虚拟机。

### 4.7. 在虚拟机中配置网络

1. 在虚拟机中，设置一个静态IP（如 `192.168.100.2`），子网掩码为 `255.255.255.0`，网关为 `192.168.100.1`。

### 4.8. 测试连接

在虚拟机中，尝试 ping 外部地址，如 `ping 192.168.0.1`以确认能访问内网路由器，`ping www.baidu.com`以确保能访问互联网。

## 五、修复windows引导

### 5.1.修改grub配置

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

### 5.2.grub页面手动测试引导

在 GRUB 启动时，按下 c 进入命令行，手动尝试引导 Windows：

```bash
set root=(hd0,gpt1)
chainloader /EFI/Microsoft/Boot/bootmgfw.efi
boot
```

如果报错，可以尝试修改一下 `root` 参数，如`hd0` 修改 `hd1`，再尝试。
