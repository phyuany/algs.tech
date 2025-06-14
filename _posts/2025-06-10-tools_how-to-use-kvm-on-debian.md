---
title: 怎样在Debian里使用kvm虚拟机
date: 2025-06-14 17:09:00 +0800
categories: [工具]
tags: [kvm]
pin: false
---

## 一、安装

### 1.1.确认kvm是否已经启用

首先查看kvm是否正常启用，如下命令

```shell
lsmod | grep kvm
```

如果显示以下类似的信息，则表示kvm已经启用，可以继续安装

```shell
kvm_intel             380928  0
kvm                  1146880  1 kvm_intel
irqbypass              16384  1 kvm
```

现代的操作系统已经默认开启了kvm功能。

### 1.2.安装工具包

```shell
apt install qemu-system libvirt-daemon-system virt-manager
```

### 1.3.启用libvirtd

```shell
systemctl enable libvirtd
systemctl start libvirtd
```

libvirtd是libvirt的守护进程，它负责管理虚拟机，启动、停止、配置等，即虚拟化工具的统一接口。

### 1.4.启动默认网路

在 `libvirt` 中，虚拟机需要连接到虚拟网络，而默认网络 (`default`) 在安装后没有自动激活。

可以通过以下步骤来激活默认网络，运行以下命令以启动并激活 `default` 网络

```bash
virsh net-start default
```

为了避免每次启动虚拟机时手动启动网络，可以设置 `default` 网络为开机自动启动：

```bash
virsh net-autostart default
```

可以使用以下命令查看当前网络的状态，确保 `default` 网络处于 "active" 状态：

```bash
virsh net-list --all
```

输出中应该看到如下类似的结果，代表默认网络已启用

```shell
 名称       状态    自动启动   持久
--------------------------------------
 default    active   yes         yes
```

## 二、创建虚拟机

### 2.1.创建虚拟机空盘

根据自己电脑的情况，创建一个目录，我这里存放在`/opt/kvm`之下，执行如下命令

```shell
# 创建目录
mkdir -p /opt/kvm
# 创建一个虚拟机空盘
qemu-img create -f qcow2 /opt/kvm/debian12-base.qcow2 30G
```

### 2.2.通过iso镜像文件创建虚拟机

先创建一个目录 `/var/libvirt/qemu/`，因为接下来虚拟机需要使用到该目录。

```shell
virt-install \
--osinfo=debian11 \
--name=debian12-base \
--virt-type=kvm \
--ram=2048 \
--vcpus=2 \
--cdrom=/opt/kvm/iso/debian-12.11.0-amd64-DVD-1.iso \
--boot=hd \
--disk path=/opt/data/kvm/debian12-base.qcow2 \
--channel unix,path=/var/libvirt/qemu/debian12-base.agent,mode=bind,target_type=virtio,name=org.qemu.guest_agent.0 \
--network network=default \
--graphics vnc,listen=0.0.0.0,port=5900 \
--noautoconsole
```
