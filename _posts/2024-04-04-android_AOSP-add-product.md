---
title: 在AOSP中添加Product
date: 2024-04-04 11:48:00 +0800
categories: [Android]
tags: []
pin: false
---

## 一、概述

在编译系统的时候，我们需要执行 `lunch` 命令，来选择编译的 `product`。在APP开发中，一份源码可以编译出多个不同的渠道包，而在编译系统中，我们通过 `lunch` 命令来选择编译的 `product`。`product`存在的作用是使用同一份源码通过不同的配置文件，来编译成不同的镜像，最终用于不同的硬件产品。

## 二、product配置文件

### 2.1 product 文件

product 配置文件所在目录如下：

- `build/target`: 模拟器相关的 product 文件
- `device`: 实际硬件设备相关的 product 文件，主要是 pixel 手机相关的 product 文件

以 `aosp_x86_64-eng` 这个模拟器为例，相关product文件如下：

- `build/target/board/generic_x86_64/BoardConfig.mk`：用于定义和硬件相关的底层特性和变量
- `build/target/product/AndroidProducts.mk`
- `build/target/product/aosp_x86_64.mk`

### 2.2 AndroidProducts.mk

在`build/target/board/generic_x86_64/BoardConfig.mk`文件中存在以下的代码：

```shell
include build/make/target/board/BoardConfigGsiCommon.mk
include build/make/target/board/BoardConfigEmuCommon.mk
```

代表引入了 `BoardConfigGsiCommon.mk` 和 `BoardConfigEmuCommon.mk` 文件，如果这两个文件不存在，编译会报错。假设我们希望引入不到文件时不报错，可以使用 `-include` 命令。

### 2.3 AndroidProducts.mk

`build/target/product/AndroidProducts.mk`定义了很多的product。我们在执行编译命令的时候，可以通过 `lunch` 命令来选择编译的product。

### 2.4 aosp_x86_64.mk

`build/target/product/aosp_x86_64.mk` 文件时定义产品信息的主要配置文件，也是修改最多的文件。

其中有几个定义产品名称的变量如下：

```shell
PRODUCT_NAME := aosp_x86_64
PRODUCT_DEVICE := generic_x86_64
PRODUCT_BRAND := Android
PRODUCT_MODEL := AOSP on x86_64
```

另外，有一个`call inherit-product`关键词，这是自定一个Makefile的一个函数，作用与`include`比较类似。配置文件中关键部分如下：

```shell
# GSI for system/product
$(call inherit-product, $(SRC_TARGET_DIR)/product/core_64_bit.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/gsi_common.mk)

# Emulator for vendor
$(call inherit-product-if-exists, device/generic/goldfish/x86_64-vendor.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/emulator_vendor.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/board/generic_x86_64/device.mk)
```

`include` 和`inherit-product` 的区别如下

- 假设 `PRODUCT_VAR:=a` 在`A.mk`中，`PRODUCT_VAR:=b`在`B.mk`
- 如果你在 `A.mk` 中`include B.mk`，你最终会得到 `PRODUCT_VAR;=b`。
- 但是如果你在 `A.mk inherit-product B.mk`，你会得到 `PRODUCT_VAR：=ab`。`inherit-product`确保您不会两次包含同一个makefile。

有另一个操作`call inherit-product-if-exists` 代表如果文件存在，则执行。

## 三、添加一个product

### 3.1 添加product文件

假设我们的公司名叫做`jelly`，要开发的产品名称是`rice14`，需要创建一个文件目录，如下命令

```shell
mkdir -p device/jelly/rice14
```

同时在目录下添加三个文件，文件名如下：

```shell
device/jelly/rice14/BoardConfig.mk
device/jelly/rice14/AndroidProducts.mk
device/jelly/rice14/rice14.mk
```

其中，`BoardConfig.mk`和`rice14.mk`相关配置内容我们直接复用 `aosp_x86_64` 的配置。

### 3.2 修改rice14.mk

在`rice14.mk`配置文件中，我们修改其中的呢绒，直接加载`PRODUCT_ENFORCE_ARTIFACT_PATH_REQUIREMENTS := relaxed`，去掉其if判断。最终代码如下：

```makefile
PRODUCT_ENFORCE_ARTIFACT_PATH_REQUIREMENTS := relaxed
```

同时修改`PRODUCT_NAME`变量，最终如下

```shell
PRODUCT_NAME := rice14
PRODUCT_DEVICE := generic_x86_64
PRODUCT_BRAND := Android
PRODUCT_MODEL := AOSP on x86_64
```

### 3.3 修改AndroidProducts.mk

另外需要在`AndroidProducts.mk`文件中，添加 product 选项。新增内容如下：

```mk
PRODUCT_MAKEFILES := \
    $(LOCAL_DIR)/rice14.mk

COMMON_LUNCH_CHOICES := \
    rice14-eng \
    rice14-userdebug \
    rice14-user
```

### 3.4 运行测试

最后在源码目录执行`lunch`，如果出现如下product选项，则成功。

```shell
44. rice14-eng
45. rice14-user
46. rice14-userdebug
```

我们选择`44`，如果吗没有出现报错，代表product添加成功。
