---
title: 在AOSP源码添加二进制可执行程序
date: 2024-04-04 15:23:00 +0800
categories: [Android]
tags: []
pin: false
---

## 一、概述

要在AOSP源码中添加二进制可执行程序，需我们需要知道以下几个目录

```shell
/system
/vendor
/odm
/product
```

同时需要知道Android硬件产品（电视、手机、平板）开发的常规流程如下：

- 1. Google 开发和迭代 AOSP + Kernel
- 2. 芯片厂商，针对自己的芯片特点，移植 google 开源的 AOSP + Kernel，使其在自己的芯片上跑起来
- 3. 方案厂商（很多芯片厂商也扮演了方案厂商的解决），设计电路板，添加外设。软件上主要是开发硬件驱动和hal，实力强的厂商还会修改部分bug和做一些性能优化。
- 4. 产品厂商，通常就是 odm oem 厂商，主要是做系统软件开发。实力强的厂商可能还会修改主板，添加自己的外设和芯片，软件上开发自己的驱动和hal程序。另外，修改bug和性能优化是产品厂商的永远干不完的工作。

产品厂商最终写的代码，大多会写到`/product`和`/odm`分区。

## 二、添加C++源码

### 2.1 添加源代码文件

首先我们需要在我们定义的`product`目录下新增一个`helloworld`目录，然后在该目录下添加一个`helloworld.cpp`文件，内容如下：

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World! Android C/C++" << std::endl;
    return 0;
}
```

### 2.2 添加Android.bp文件

该文件用于描述编译规则，在`helloworld`目录下新增一个`helloworld.bp`文件，内容如下：

```bp
cc_binary {
    name: "helloworld",
    srcs: ["helloworld.cpp"]
}
```

### 2.3 单模块编译

首先我们执行单模块编译，命令如下

```shell
# 执行初始化
source build/envsetup.sh
# 选择编译目标
lunch rice14-eng
# 进入源码目录
cd device/jelly/rice14/helloworld/
# 执行编译
mm
```

编译成功后，终端将输出如下信息

```shell
15:47:32 Build configuration changed: "aosp_x86_64-eng" -> "rice14-eng", forcing installclean
[100% 23/23] Install: out/target/product/generic_x86_64/system/bin/helloworld

#### build completed successfully (01:13 (mm:ss)) ####
```

### 2.4 整体编译

#### 2.4.1 将可执行文件添加到system分区

首先需要在product目录的`rice14.mk`添加如下一行两个关键变量，分别用于打包可执行文件到系统和添加可执行文件到system分区的白名单。

```shell
# 添加helloworld
PRODUCT_PACKAGES += helloworld

# 添加可执行文件到系统白名单
PRODUCT_ARTIFACT_PATH_REQUIREMENT_WHITELIST += \
    system/bin/helloworld

# 原产品名称变量
PRODUCT_NAME := rice14
PRODUCT_DEVICE := generic_x86_64
PRODUCT_BRAND := Android
PRODUCT_MODEL := AOSP on x86_64
```

执行整体便利，命令如下

```shell
# 执行初始化
source build/envsetup.sh
# 选择编译目标
lunch rice14-eng
# 编译
m
```

#### 2.4.2 将可执行文件添加到product分区

不过我们需要注意的是，不管是做芯片、方案、产品厂商，都尽量避免添加到`/system`目录下。通常是放在 `product` 分区，因此需要在`Android.bp`添加`product_specific: true`，最终代码如下：

```bp
cc_binary {
    name: "helloworld",
    srcs: ["helloworld.cpp"],
    product_specific: true
}
```

同时需要把`rice14.mk`的如下代码删除

```shell
# 添加可执行文件到系统白名单
PRODUCT_ARTIFACT_PATH_REQUIREMENT_WHITELIST += \
    system/bin/helloworld
```

再执行一次整体编译，命令如下

```shell
m
```

最终编译完成后，终端将输出如下信息

```shell
The new table will be used at the next reboot.
The operation has completed successfully.
out/host/linux-x86/bin/sgdisk --clear out/target/product/generic_x86_64/system-qemu.img

#### build completed successfully (01:47 (mm:ss)) ####
```

### 2.5 验证可执行程序

编译完成后，使用`emulator`命令启动模拟器，并在终端使用`adb shell`命令打开安装shell，查看`/product/bin`目录下是否存在`helloworld`可执行程序，可以看到如下图

![20240404161906](/img/android/20240404161906.png)

如果正常执行，则代表操作成功。

### 2.6 总结

系统源码中的二进制程序一般预制到product分区，需要在`Android.bp`中添加`product_specific: true`。

如果我们一定要将二进制程序添加到system分区，则需要在`Android.bp`中添加`product_specific: false`，同时需要在`rice14.mk`中通过`PRODUCT_ARTIFACT_PATH_REQUIREMENT_WHITELIST`变量设置白名单。

## 三、添加可执行文件

### 3.1 准备二进制可执行文件

有时候我们需要将可执行文件直接添加到系统源码中，而不是通过添加源代码最后编译生成。接下来我们使用 `busybox` 添加到系统源码中。我们先在product目录新建一个`prebuild`目录，代表编译好的二进制文件目录。再在`prebuild`目录下新建一个`busybox`目录，将编译好的二进制文件`busybox`拷贝到该目录下。以上过程执行命令如下

```shell
mkdir -p device/jelly/rice14/prebuild/busybox
cd device/jelly/rice14/prebuild/busybox
wget https://busybox.net/downloads/binaries/1.30.0-i686/busybox
```

在同级目录添加`Android.bp`文件，内容如下：

```shell
cc_prebuilt_binary {
    name: "busybox",
    srcs: ["busybox"],
    product_specific: true
}
```

接下来在`rice14.mk`中添加如下代码：

```shell
PRODUCT_PACKAGES += helloworld \
    busybox
```

### 3.2 执行整体编译

命令如下

```shell
source build/envsetup.sh
lunch rice14-eng
m
```

编译完成后，关闭虚拟机，再重新使用`emulator`启动虚拟机，使用`adb shell`命令查看`/product/bin`目录下是否存在`busybox`可执行程序，可以看到如下图`

![20240404170257](img/20240404170257.jpg)

结果如图所示则代表操作成功。

## 四、添加Java源码

### 4.1 准备Java源码

在product目录新增一个`hellojava`目录，首先建包的路径`tech.webcoding.main`，使用以下命令创建目录

```shell
mkdir -p device/jelly/rice14/hellojava/tech/webcoding/main
```

在`tech.webcoding.main`目录下新建一个`HelloJava.java`文件，内容如下：

```java
package tech.webcoding.main;

public class HelloJava {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

另外添加`Android.bp`文件，内容如下

```bp
java_library {
    name: "hellojava",
    installable: true,
    product_specific: true,
    srcs: ["**/*.java"],
    sdk_version: "current"
}
```

需要注意的是，如果不指定`installable: true`，那么编译产物都是`.class`文件，这种文件在Android无法直接运行。最后需要在`rice14.mk`中添加如下代码：

```shell
PRODUCT_PACKAGES += helloworld \
    busybox \
    hellojava
```

### 4.2 执行编译并验证

接着执行编译

```shell
source build/envsetup.sh
lunch rice14-eng
m
```

编译完成之后运行模拟器，并使用`adb shell`打开终端，查看`product/framework/`目录下是否存在`hellojava.jar`，如果存在则执行以下命令

```shell
# 配置 classpath
export CLASSPATH=/product/framework/hellojava.jar
# 执行程序
app_process /product/framework/ tech.webcoding.main.HelloJava
```

正常输出结果代表操作成功。

## 五、添加Jar包

在product目录新增一个`hellojavajar`目录，我们直接把第四步编译出的`hellojava.jar`拷贝到`hellojavajar`目录下，如下命令

```shell
cp out/target/product/generic_x86_64/system/product/framework/hellojava.jar device/jelly/rice14/hellojavajar/
```

然后添加`Android.bp`文件，内容如下：

```shell
java_import {
    name: "hellojavajar",
    installable: true,
    jars: ["hellojava.jar"],
    product_specific: true
}
```

为了避免冲突，我们需要把第四步的`hellojava`删除掉。再在`rice14.mk`中添加如下代码变更`PRODUCT_PACKAGES`变量如下

```shell
PRODUCT_PACKAGES += helloworld \
    busybox \
    hellojavajar
```

再执行编译，命令如下

```shell
source build/envsetup.sh
lunch rice14-eng
m
```

编译完成后，重新启动模拟器，再使用`adb shell`打开终端，查看`product/framework/`目录下是否存在`hellojavajar.jar`，如果存在则执行以下命令

```shell
# 配置 classpath
export CLASSPATH=/product/framework/hellojavajar.jar
# 执行程序
app_process /product/framework/ tech.webcoding.main.HelloJava
```

正常输出结果代表操作成功。
