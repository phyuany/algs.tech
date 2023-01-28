---
title: 在Ubuntu上快速搭建k8s集群
date: 2023-01-28 14:51:00 +0800
categories: [容器与虚拟化]
tags: [k8s]
pin: false
---

> 撰写时间：2021-01-17，整理时间：2023-01-28

etcd可以用于k8s集群的数据库，下面是在内网安装etcd集群的详细步骤

## 一、为etcd颁发SSL证书

### 1.1 颁发步骤

加密证书我们可以向证书机构（CA）申请，但是由于我们部署在内网，我们自己自己创建一个CA后给自己颁发证书，步骤如下

- （1）创建证书颁发机构
- （2）填写表单--写明etcd所在节点的IP
- （3）向证书颁发结构申请证书

### 1.2 安装cfssl

从cfssl官网中选择对应的cfssl相关可执行程序，[https://pkg.cfssl.org/](https://pkg.cfssl.org/)
这里直接使用wget下载保存到/usr/local/bin目录中

```shell
# 下载cfssl
wget -c https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/local/bin/cfssl
# 下载cfssl-certinfo
wget -c https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/local/bin/cfssl-certinfo
# 下载cfssljson
wget -c https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/local/bin/cfssljson
# 赋予cfssl相关可执行权限
chmod a+x /usr/local/bin/cfssl*
```

### 1.3 颁发证书

#### 1.3.1 创建签证机构（CA）

定义一个ca-csr.json文件，此文件定义签证机构（CA）的相关信息，填入以下内容

```json
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Guizhou",
            "ST": "Qiandongnan"
        }
    ]
}
```

使用以下命令使用配置文件中，创建一个签证机构

```shell
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

会生成一个ca.pem公钥文件和ca-key.pem私钥文件，这两个文件是CA服务器的证书。接下来我们需要给客户端颁发证书。

#### 1.3.2 定义证书信息

先创建一个ca-config.json文件，定义待颁发证书的基本信息（如有效时间），填写以下内容

```json
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "www": {
         "expiry": "87600h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
```

再创建一个server-csr.json文件，定义待颁发证书的详细信息，如下

```json
{
    "CN": "etcd",
    "hosts": [
        "192.168.56.101",
        "192.168.56.102",
        "192.168.56.103"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "L": "Guizhou",
        "ST": "Qiandongnan"
    }]
}
```

相关参数

- CN：证书名称
- hosts：颁发的主机
- key：定义证书类型，algo为加密类型，size为加密长度
- names：证书的基本信息

#### 1.3.3 向签证机构（CA）申请证书

完成以上步骤，使用以下命令给客户端申请证书

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server
```

此时，多出客户端两个证书相关文件server.pem和server-key.pem，使用```ls *pem```命令查看，可以看到四个pem文件，如下

```shell
ca-key.pem  ca.pem  server-key.pem  server.pem
```

## 二、手动编译安装最新版本etcd

我们从源码安装时，将安装最新的版本，包括步稳定的预览版本，此安装方式供大家学习，不见建议在生产环境中使用最新版本。

### 2.1 安装golang

etcd在构建的时候需要golang环境，所以先安装golang，使用以下命令

```shell
# 添加golang镜像仓库
rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO
curl -s https://mirror.go-repo.io/centos/go-repo.repo | sudo tee /etc/yum.repos.d/go-repo.repo
# 安装golang
yum install golang -y
# 使用以下命令查看go安装版本
go version
```

如果出现以下内容，则证明安装成功

```shell
go version go1.15.6 linux/amd64
```

此时，我们需要设置goproxy阿里云加速镜像，编辑/etc/profile文件，在末尾添加以下内容

```shell
export GOPROXY=https://mirrors.aliyun.com/goproxy/
```

并让/etc/profile里面的命令重新执行，使用以下命令

```shell
source /etc/profile
```

### 2.2 编译安装etcd

首先从github中下载源代码包，并执行构建命令

```shell
git clone https://github.com/etcd-io/etcd.git
cd etcd
./build
```

最终若是输出类似以下内容，则证明构建成功

```shell
stderr: go: downloading github.com/urfave/cli v1.22.4                                     
stderr: go: downloading github.com/bgentry/speakeasy v0.1.0
stderr: go: downloading gopkg.in/cheggaaa/pb.v1 v1.0.28
stderr: go: downloading github.com/olekukonko/tablewriter v0.0.4
stderr: go: downloading github.com/mattn/go-runewidth v0.0.7
stderr: go: downloading github.com/cpuguy83/go-md2man/v2 v2.0.0
stderr: go: downloading github.com/russross/blackfriday/v2 v2.0.1
stderr: go: downloading github.com/shurcooL/sanitized_anchor_name v1.0.0
SUCCESS: etcd_build
```

最终编译构建成功之后生成的文件在源码目录下的bin目录，使用```ls -al bin/```命令可以查看到生成了etcd和etcdctl两个可执行文件

```shell
总用量 52112
drwxr-xr-x.  2 root root       33 1月  15 10:16 .
drwxr-xr-x. 19 root root     4096 1月  15 10:15 ..
-rwxr-xr-x.  1 root root 29862882 1月  15 10:15 etcd
-rwxr-xr-x.  1 root root 23492302 1月  15 10:16 etcdctl
```

此时我们可以把etcd可执行程序拷贝到系统中

```shell
cp bin/etcd* /usr/local/bin
```

## 三、从etcd二进制文件包安装etcd

etcd官方提供直接下载二进制文件进行安装的方式。如果使用二进制安装包安装，可以不安装golang，下载官方提供的二进制程序包进行解压后，可直接得到方式一中的etcd和etcdctl。下载地址为

[https://github.com/etcd-io/etcd/releases/](https://github.com/etcd-io/etcd/releases/)

后续步骤和方式一一样，即先创建etcd目录，再把etcd和etcdctl两个文件拷贝到对应目录即可。

在写这篇文章的时候，etcd最新的稳定版本为3.4.14，所以我们指定3.4.14版本进行下载

```shell
# 下载二进制程序包
wget https://github.com/etcd-io/etcd/releases/download/v3.4.14/etcd-v3.4.14-linux-amd64.tar.gz
# 解压程序
tar -zxvf etcd-v3.4.14-linux-amd64.tar.gz
# 复制二进制文件到系统
cp etcd-v3.4.14-linux-amd64/etcd* /usr/local/bin/
```

## 四、配置etcd服务管理脚本

我们已经将etcd安装到系统中，接下来我们需要编写etcd的systemd服务管理脚本，用systemd来管理etcd的启动。systemd知识属于Linux的系统知识，我们只介绍本次用到的知识点，其他部分在这里不会赘述。在Linux系统中，systemd服务管理脚本在以下两个目录

- 系统脚本目录

系统默认的服务脚本目录

```shell
/etc/systemd/system
```

- 用户脚本目录

用户自定义脚本，通常保存在这个目录

```shell
/usr/lib/systemd/system
```

创建一个systemd服务管理脚本```/usr/lib/systemd/system/etcd.service```，并在文件中定义内容。下文介绍启动etcd的两种方式。

### 4.1 指定环境变量启动etcd

在服务脚本文件中添加以下内容

```ini
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/etc/etcd/conf/etcd.conf
ExecStart=/usr/local/bin/etcd \
        --cert-file=/etc/etcd/ssl/server.pem \
        --key-file=/etc/etcd/ssl/server-key.pem \
        --peer-cert-file=/etc/etcd/ssl/server.pem \
        --peer-key-file=/etc/etcd/ssl/server-key.pem \
        --trusted-ca-file=/etc/etcd/ssl/ca.pem \
        --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

syetemd参数配置文件的Unit节点用来定义元数据，Service节点用来配置服务，Install节点是配置文件的最后一个区块，用来定义服务启动方式

- [Unit] **Description**：服务描述
- [Unit] **After**：当前Unit需要在哪些Unit启动之后启动
- [Unit] **Wants**：与当前 Unit 配合的其他 Unit，如果其他Unit没有运行，当前 Unit 也不会启动失败

- [Service] **Type**：定义启动进程的行为，值为notify时，代表当前服务启动完毕，会通知`Systemd`，再继续往下执行
- [Service] **EnvironmentFile**：指定环境变量配置文件
- [Service] **ExecStart**：启动当前服务的命令
- [Service] **Restart**：定义何种情况 Systemd 会自动重启当前服务，on-failure代表服务停止后会重启
- [Service] **LimitNOFILE**：设置资源限制，即用户最多可以使用多少个进程

- [Install] **WantedBy**：当值为multi-user.target时，代表系统大部分默认服务启动了（不包含GUI），服务跟着启动。

在systemd服务配置文件中，我们最需要关注的是[Service] ExecStart，这个参数定义systemd去哪里找etcd的启动程序，我们需要指定etcd具体的二进制文件所在路径。我们还指定了etcd运行时的环境变量配置文件，我们需要在对应的路径（/etc/etcd/conf/etcd.conf）创建其对应文件，并添加以下内容

```shell
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.56.101:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.56.101:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.56.101:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.56.101:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.56.101:2380,etcd-2=https://192.168.56.102:2380,etcd-3=https://192.168.56.103:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```

在etcd的环境变量配置文件中，包含两段，[Member]之下是单节点配置参数，[Clustering]节点下是集群配置参数

- [Member] **ETCD_NAME**：当前etcd的唯一名称，要保证和其他节点不冲突

- [Member] **ETCD_DATA_DIR**：指定etcd存储数据的存储位置

- [Member] **ETCD_LISTEN_PEER_URLS**：指定当前etcd和其他节点etcd通信时的服务地址和端口。假如etcd集群中包含三个etcd服务，那么三个etcd节点构成了一个高可用集群，etcd集群会自动选举一个节点作为主节点，另外两个节点作为从节点。所有的写入操作将往主节点写，所有的读操作在从节点上读，这是etcd读写的过程。上述设置的2380端口，用来监听其他etcd节点发送过来的数据。

- [Member] **ETCD_LISTEN_CLIENT_URLS**：指定当前etcd接收客户端指令的地址和端口，在这里的客户端我们指的是k8s集群的master节点。

- [Clustering] **ETCD_INITIAL_ADVERTISE_PEER_URLS**：指定etcd广播端口，当前etcd会将数据同步到其他节点，通过2380端口发送。
- [Clustering] **ETCD_ADVERTISE_CLIENT_URLS**：给客户端通告的端口。
- [Clustering] **ETCD_INITIAL_CLUSTER**：定义etcd集群中所有节点的名称和IP，以及通信端口。
- [Clustering] **ETCD_INITIAL_CLUSTER_TOKEN**：定义etcd集群中通信的token，所有节点必须一致。
- [Clustering] **ETCD_INITIAL_CLUSTER_STATE**：定义etcd集群的状态，new代表新建集群，existing代表加入现有集群。

### 4.2 指定参数启动etcd

如果我们指定参数启动etcd，则不需要指定环境配置变量文件，这两种方式二选一即可

```shell
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
        --name=${ETCD_NAME} \
        --data-dir=${ETCD_DATA_DIR} \
        --listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
        --listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
        --advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
        --initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
        --initial-cluster=${ETCD_INITIAL_CLUSTER} \
        --initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
        --initial-cluster-state=new \
        --cert-file=/etc/etcd/ssl/server.pem \
        --key-file=/etc/etcd/ssl/server-key.pem \
        --peer-cert-file=/etc/etcd/ssl/server.pem \
        --peer-key-file=/etc/etcd/ssl/server-key.pem \
        --trusted-ca-file=/etc/etcd/ssl/ca.pem \
        --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

> 注意：在systemd服务配置文件中，我们指定了etcd的SSL证书路径，我们需要把申请的etcd的SSL证书复制到对应的目录。另外，每个etcd节点的配置文件里的etcd名称一定要唯一，对应的服务IP与端口也不能出错。在k8s-master1节点安装好之后，k8s-node1和k8s-node2需要进行相同操作，SSL证书需要从k8s-master1节点上传输过去进行安装。

## 五、启动etcd集群

所有节点都安装好etcd之后，每个节点一次执行以下命令

```shell
# 启动etcd
systemctl start etcd
# 设置开机自启
systemctl enable etcd
```

启动完成之后，我们在任意节点使用etcdctl命令检查集群状态，需要注意的是，要确切指定证书的位置

```shell
etcdctl --cacert="/etc/etcd/ssl/ca.pem" --cert="/etc/etcd/ssl/server.pem" --key="/etc/etcd/ssl/server-key.pem" --endpoints="https://192.168.56.101:2379,https://192.168.56.102:2379,https://192.168.56.103:2379" endpoint health --write-out=table
```

如果显示一下信息，则etcd集群服务正常；如果不显示以下内容，则需要重新检查安装的每一个步骤，再来一次确保无误

```shell
+-----------------------------+--------+-------------+-------+
|          ENDPOINT           | HEALTH |    TOOK     | ERROR |
+-----------------------------+--------+-------------+-------+
| https://192.168.56.101:2379 |   true | 33.529463ms |       |
| https://192.168.56.103:2379 |   true | 38.145028ms |       |
| https://192.168.56.102:2379 |   true | 34.107771ms |       |
+-----------------------------+--------+-------------+-------+
```
