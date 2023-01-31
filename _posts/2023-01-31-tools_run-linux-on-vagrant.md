---
title: 怎样使用Vagrant在命令行终端命上管理Linux虚拟机？
date: 2023-01-31 21:41:00 +0800
categories: [工具]
tags: [虚拟机 vagrant]
pin: false
---

## 一、Vgrant的安装

Vagrant是一个跨平台的虚拟机管理工具，我们以 Deepin 20.2.3 为例，安装和使用 Vagrant。我们在这里所说的 Vagrant 包括 Vagrant 工具本身 和 虚拟引擎工具 VirtualBox。

### 1.1 安装Vagrant

安装 vagrant 时，使用的 VirtualBox 版本必须要得到 对应 Vagrant 版本的支持，在写这篇文档的时候，我安装的 Vagrant 版本是 `2.2.18` ，对应的 VirtualBox 版本是 `6.1.26`。

Vagrant 安装包下载地址：[https://releases.hashicorp.com/vagrant/](https://releases.hashicorp.com/vagrant/)，在这里可以看到Vagrant 当前的最新版本以及历史版本，我们可以选择到对应操作系统的对应版本进行下载。

VirtualBox 安装包下载地址：[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)，同样，在这里我们可以找到 VirtualBox 的对应版本。

将 Vagrant 和 VirtualBox 指定的安装包下载到本地后，执行安装命令，如下

```shell
# 安装 Vagrant
sudo apt install ./vagrant_2.2.18_x86_64.deb
# 安装 VirtualBox
sudo apt install ./virtualbox-6.1_6.1.26-145957_Debian_buster_amd64.deb
```

### 1.2 安装验证

执行以下命令验证vagrnat是否能正常使用

```shell
# 创建 vagrant 项目路径
mkdir ~/my-vagrant-project
# 初始化 centos8虚拟机
vagrant init centos/8
# 启动虚拟机
vagrant up
# 停止虚拟机
vagrant halt
# 删除虚拟机
vagrant destroy
```

如果以上步骤可以顺利执行，则代表 Vagrant 安装成功。

## 二、Vagrant的使用

### 2.1 安装 box

我们可以把 Vgarnt 中的 `box` 理解为某种操作系统的镜像文件，也可以理解为虚拟机的本身，我们可以去添加我们想要的 `box`， Vagrant 可以去管理这些 `box`，我们在启动虚拟机时可以选择我们想要的 `box`。

我们可以通过[https://app.vagrantup.com/boxes/search](https://app.vagrantup.com/boxes/search)去找到我们想要的box，我们可以看到 box 名称格式为 `A/B`， 其中 A 代表创建 box 的用户名，B 为 box 的名称，如`centos/7`。

- 安装指定的 box

```shell
vagrant box add centos/7
```

此时看到如下选择提示

```shell
pan@pan-PC:~$ vagrant box add centos/7
==> box: Loading metadata for box 'centos/7'
    box: URL: https://vagrantcloud.com/centos/7
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 
```

因为我们使用的虚拟机引擎是`virtualbox`，所以输入`3`后按回车即下载 virtualbox 版本的 centos/7。

- 查看已经安装的box

```shell
vagrant box list
```

### 2.2 虚拟机的基本使用

- 初始化虚拟机

```shell
# 创建一个空项目目录
mkdir ~/test
# 进入项目目录
cd ~/test
# 使用centos/7初始化vagrant项目
vagrant init centos/7
```

此时在目录下生成一个名为 `Vagrantfile` 的配置文件，在后续，我们可以通过修改 `Vagrantfile` 来定义虚拟机，接下来的相关操作指令都将在 vagrant 项目目录下执行

- 启动虚拟机

```shell
vagrant up
```

虚拟机启动之后，会将本地的项目目录自动挂载到虚拟机里的 `/vagrant` 目录

- 连接虚拟机

```shell
# 默认使用vagrant用户连接到虚拟机
vagrant ssh
```

另外，我们可以通过`vagrant ssh-config`命令查看ssh配置信息，包括虚拟机与本地的映射端口、证书文件等，如下

```shell
pan@pan-PC:~/Work/vagrant/centos$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/pan/Work/vagrant/centos/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

我们也可以通过直接通过 ssh 工具连接到虚拟机，如下命令

```shell
ssh vagrant@127.0.0.1 -p 2222 -i .vagrant/machines/default/virtualbox/private_key
```

vagrant用户的默认密码为`vagrant`

相关参数参数

**-p**：指定ssh端口

**-i**：指定ssh私钥文件

- 查看虚拟机状态

```shell
vagrant status
```

- 虚拟机关机

```shell
vagrant halt
```

- 暂停虚拟机

```shell
# 再次启动虚拟机
vagrant up
# 退出虚拟机
exit
# 暂停虚拟机
vagrant suspend
# 继续运行虚拟机
vagrant resume
```

vagrant 将会在虚拟机暂停之后，把数据存储到我们本地硬盘，下次开启时，将从硬盘读取恢复。在虚拟机暂停的过程中，虚拟机相关服务的状态会进行保存，如`httpd` 服务的正在运行，虚拟机暂停后再继续运行虚拟机，`httpd` 服务也会继续运行

- 重启虚拟机

在项目目录下，执行`vagrant reload`，虚拟机将会先执行关机动作，后执行开机动作。

- 销毁当前的虚拟机

```shell
vagrant destroy
```

销毁虚拟机操作会将虚拟机进行关机后删除实例，如果虚拟机已经处于关机状态将直接删除。此命令只会销毁我们在当前目录创建的 虚拟机，不会销毁 box。
此时我们使用 `vagrant status` 命令查看虚拟机状态，会看到虚拟机处于 `not created` 状态。

## 三. 用Vagrantfile定义虚拟机

### 3.1 添加额外共享目录

实际上，在虚拟机创建的时候，vagrant 会将项目目录自动挂载到 虚拟机里的 `/vagrant` 目录，实际上，我们还可以通过 定义 `Vagrantfile` 的方式来挂载其他目录。

打开 `Vagrantfile` 找到 `# config.vm.synced_folder "../data", "/vagrant_data"` 所在的行，去掉注视符号 “#”，即定义了一个目录挂载规则，第一个参数代表 宿主机当前目录的上级目录下 的 `data` 目录， 第二个参数 代表挂载 到虚拟机里的 `/vagrant_data` 目录 。我们还可以添加后面的参数，最终代码如下

```ruby
config.vm.synced_folder "../data", "/vagrant_data", create: true, owner: "root", group: "root"
```

**create: true**：代表如果宿主机不存在对应的目录，将会创建；

**owner: "root"**：代表挂载到虚拟机后，目录的拥有者是 root；

**group: "root"**：代表挂载到虚拟机后，目录的所属的群组是 root。

保存好`Vagrantfile`之后，执行`vagrant reload`即完成虚拟机的重启并挂载上对应的目录。如果执行报错的话，我们需要使用`vagrant plugin install vagrant-vbguest --plugin-version 0.21` 安装vbguest插件后再次重启虚拟机。

### 3.2 网络配置

我们通常需要跟虚拟机进行通信，比如我们在虚拟机上安装了一个web服务，需要通过自己电脑的浏览器打开虚拟机上搭建的web服务。这就需要我们配置虚拟机的网络，宿主机才能与虚拟机正常通信。vagrant 提供三种网络的配置。

- 端口转发（forwarded_port）
如把宿主机的 8080 端口，转发到虚拟机的 80 端口，这样 如果在宿主机访问 `http://localhost:8080` 将对转发到虚拟机的 80 端口服务。这种方法不太灵活，因为我们需要配置所有需要转发的端口。

- 私有网络（private_network）
为虚拟机手动设置IP地址，通过IP地址我们的宿主机就可以与虚拟机之间通信了，不过，我们只能通过我们的宿主机访问虚拟机。

- 公有网络（public_network）
公有网络会把虚拟机配置成在同一个局域网内可以访问的一台设备。比如我我们的电脑链接一台路由器，那么其他连接这台路由器的设备（其他电脑、平板电脑、手机等）也可以访问到我们电脑里的虚拟机。

- 配置私有网络

找到 `config.vm.network "private_network"` 所在行，取消注释，第2个参数是指定的ip地址

```ruby
config.vm.network "private_network", ip: "192.168.33.10"
```

接下来用`ping 192.168.33.10`命令去验证，如果有返回，代表私有网络设置成功。

- 配置共有网络

找到`config.vm.network "public_network"`所在行，取消注释，并且注释私有网络的配置。后执行`vagrant reload`重新加载 Vagrantfile 文件，此时需要选择指定的网卡。如下输出内容：

```bash
pan@pan-PC:~/Work/vagrant/ubuntu$ vagrant reload
==> default: Attempting graceful shutdown of VM...
==> default: Checking if box 'ubuntu/xenial64' version '20210804.0.0' is up to date...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Available bridged network interfaces:
1) enp3s0
2) docker0
3) vethc80d1c8
4) vethca1c3e4
5) veth3992844
6) veth2014eaa
7) vethd2642b2
8) veth2b5b670
9) veth21318b5
==> default: When choosing an interface, it is usually the one that is
==> default: being used to connect to the internet.
==> default: 
    default: Which interface should the network bridge to?
```

我们需要选择对应的网卡即可，此时我需要输入1后按回车，因为enp3s0是我电脑的实体网卡名称，而其他选项都是通过软件创建的虚拟网卡。

### 3.3 虚拟机打包

我们可以对正在运行的某个虚拟机进行打包生成镜像。这样，就可以使用新的镜像进行创建虚拟机，那么新创建的虚拟机会跟打包时的虚拟机环境一致。步骤如下：

（1）先在虚拟机删除网卡定义文件

```shell
sudo rm -rf /etc/udev/rules.d/70-persistent-net.rules
```

（2）退出虚拟机，进行打包

```shell
vagrant package
```

此时在当前目录生成`package.box`镜像文件

（3）将镜像文件导入box中

```shell
# 将导入的box命名为 jkdev/ubuntu
vagrant box add jkdev/ubuntu package.box
```

导入成功后，即可使用 `jkdev/ubuntu` 创建新的虚拟机

### 3.4 创建多台虚拟机

在实际项目中，有时候我们会把 web 服务放在一台或者多台服务器， 数据库服务器放在一台服务器。那么在本地开发中，我们也需要去模拟对应的服务器环境。如果我们可以创建多台虚拟机，这样我们就可以模拟一个真实的服务器环境。实际上， `Vagrantfile` 可以同时定义多台服务器，每一台主机都可以有自己的配置，同时我们需要把每一台主机的网络配置好，那么各台服务器之间就可以进行通信了。创建过程如下：

```ruby
# 创建目录
mkdir project
# 初始化Vagrantfile
vagrant init ubuntu/xenial64
```

在`Vagranfile` 在`config.vm.box = "ubuntu/xenial64"`下面的一行添加以下代码

```ruby
# 定义名为development的虚拟机，并启用该虚拟机
config.vm.define "development" do |development|
end

# 定义名为production的虚拟机，并启用该虚拟机
config.vm.define "production" do |production|
end
```

保存文件之后，使用`vagrant up`启动虚拟机，此时同时开启两个虚拟机。启动完毕后使用`vagrant status`，可以看到已经启动了两个虚拟机

```bash
pan@pan-PC:~/Work/vagrant/test$ vagrant status
Current machine states:

development               running (virtualbox)
production                running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
pan@pan-PC:~/Work/vagrant/test$
```

如果我们只想启动一台虚拟机，在启动命令的后面指定对应的虚拟机名称即可，如`vagrant up production`。如果我们需要连接该虚拟机，使用如下代码

```shell
vagrant ssh production
```

### 3.5 配置多台虚拟机的网络

在4中Vagrantfile文件里，我们定义了两台虚拟机，这两台虚拟机都基于`ubuntu/xenial64`，原因是该配置与定义虚拟机的配置属于同一级，如果我们想要为某台虚拟机定义独立的配置，我们需要将配置内容定义在`config.vm.define`和`end`之间。如下我们定义两个主机的私有网络，注意私有网络的IP地址不要和宿主机连接的路由器IP地址重复。如下配置

```ruby
config.vm.define "development" do |development|
  development.vm.network "private_network", ip: "192.168.33.11"
end

config.vm.define "production" do |production|
  production.vm.network "private_network", ip: "192.168.33.12" 
end
```

再使用`vagrant reload`即可完成配置的重加载。

### 3.6 定义主机名

在5中，我们一定配置好了主机的IP地址，我们接下来追加配置，定义主机的名称，如下代码：

```ruby
config.vm.define "development" do |development|
  development.vm.network "private_network", ip: "192.168.33.11"
  development.vm.hostname = "dev"
end

config.vm.define "production" do |production|
  production.vm.network "private_network", ip: "192.168.33.12" 
  production.vm.hostname = "pro"
end
```

### 3.7 自定义同步目录

默认配置下，多台虚拟机会自动共享电脑上项目所在目录，在虚拟机里会映射到`/vagrant`目录，我们还可以单独为不同虚拟机设置不同的目录，首先在项目下创建两个目录`dev`和`pro`，修改配置文件如下

```ruby
config.vm.define "development" do |development|
  development.vm.network "private_network", ip: "192.168.33.11"
  development.vm.hostname = "dev"
  # 将dev 目录挂载到/vagrant_dev目录
  development.vm.synced_folder "dev", "/vagrant_dev"
end

config.vm.define "production" do |production|
  production.vm.network "private_network", ip: "192.168.33.12" 
  production.vm.hostname = "pro"
  # 将pro 目录挂载到/vagrant_pro目录
  production.vm.synced_folder "pro", "/vagrant_pro"
end
```

### 3.8 指定cpu核心和内存

我们还可以自定义虚拟机的内存和核心数，如下代码

```ruby
config.vm.define "development" do |development|
  development.vm.network "private_network", ip: "192.168.33.11"
  development.vm.hostname = "dev"
  # 将dev 目录挂载到/vagrant_dev目录
  development.vm.synced_folder "dev", "/vagrant_dev"
  # 指定核心数和内存
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
  end
end

config.vm.define "production" do |production|
  production.vm.network "private_network", ip: "192.168.33.12" 
  production.vm.hostname = "pro"
  # 将pro 目录挂载到/vagrant_pro目录
  production.vm.synced_folder "pro", "/vagrant_pro"
  # 指定核心数和内存
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
  end
end
```

使用`vagrant reload`完成虚拟机重加载
