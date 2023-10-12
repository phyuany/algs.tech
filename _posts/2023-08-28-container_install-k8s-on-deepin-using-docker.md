---
title: 在Ubunt上快速搭建k8s集群
date: 2023-08-28 13:10:00 +0800
categories: [容器与虚拟化]
tags: [k8s]
pin: false
---

## 一、准备环境

> 撰写时间：2023-08-28

我们需要借助一个工具叫做`minicuke`，网址：<https://minikube.sigs.k8s.io/>，安装好minicube之后，我们可以安装单机的k8s环境，如下步骤

我的环境是： Deepin 20.9 (基于debian 10)

## 二、安装过程

### 2.1 删除旧环境

如果之前安装过，可以执行以下步骤

```shell
-- 删除ingress-nginx插件
minikube kubectl -- delete all --all -n ingress-nginx
-- 删除本地集群
minikube delete --all
```

### 2.2. 启动新集群

```shell
minikube start --kubernetes-version=v1.24.1 --driver=docker --container-runtime=containerd --image-mirror-country=cn --cpus=4 --memory=8g
```

#### 2.3. 新增ingress-nginx插件

```shell
minikube addons enable ingress
```

> 如果网络不顺畅，可以额外指定--images参数使用自定义镜像，自定义镜像使用教程：https://minikube.sigs.k8s.io/docs/handbook/addons/custom-images/

使用自定义镜像安装插件示例如下：

```shell
minikube addons enable ingress --images="IngressController=jikerdev/ingress-nginx-controller:v1.7.0,KubeWebhookCertgenCreate=jikerdev/kube-webhook-certgen:v20230312-helm-chart-4.5.2-28-g66a760794,KubeWebhookCertgenPatch=jikerdev/kube-webhook-certgen:v20230312-helm-chart-4.5.2-28-g66a760794" --registries="IngressController=docker.io,KubeWebhookCertgenCreate=docker.io,KubeWebhookCertgenPatch=docker.io"
````


### 2.4. 新增ingress-dns

```shell
minikube addons enable ingress-dns
```

### 2.5. 新增本地仓库服务

```shell
minikube addons enable registry
```

使用仓库服务时需要映射端口，如下命令，将宿主机 5000 映射到容器 80

```
kubectl port-forward --namespace kube-system service/registry 5000:80
```

## 三、集群验证

使用如下命令进行验证，如所有pod正常，代表安装成功

```shell
minikube kubectl -- get pod -A
```

未能能快速调用`kubectl`命令，我们还可以设置命令别名，如下命令

```shell
alias kubectl='minikube kubectl --'
```



