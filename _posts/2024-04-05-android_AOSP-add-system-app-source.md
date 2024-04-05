---
title: 在AOSP中添加系统APP
date: 2024-04-05 01:23:00 +0800
categories: [Android]
tags: []
pin: false
---

## 一、新建APP项目

我们使用`Android Studio 3.6.3`版本新建一个空的APP项目，因为该版本是`Android 10 r41`发布之后的版本，适合用于基于`Android 10`的APP开发，Android Studio 历史版本下载地址为：[https://developer.android.google.cn/studio/archive](https://developer.android.google.cn/studio/archive)

新建的APP基本信息如下如所示：

![20240405022621](/img/android/20240405022621.png)

## 二、拷贝APP项目到AOSP

接下来我们在AOSP的product目录下新建一个目录，名为`FirstSystemApp`，然后在该目录下新增`src`和`res`目录，把新建的APP项目对应的资源和代码这两个目录之下，再把`AndroidManifest.xml`文件复制过来到`FirstSystemApp`目录下。

另外需要在`FirstSystemApp`目录下新建一个`Android.bp`文件，内容如下：

```shell
android_app {
    name: "FirstSystemApp",
    srcs: ["src/**/*.java"],
    resource_dirs: ["res"],
    manifest: "AndroidManifest.xml",
    platform_apis: true,
    sdk_version: "",
    certificate: "platform",
    product_specific: true,

    static_libs: [
        "androidx.appcompat_appcompat",
        "com.google.android.material_material",
        "androidx-constraintlayout_constraintlayout",
    ]
}
```

在`rice14.mk`文件中变更`PRODUCT_PACKAGES`变量为

```shell
PRODUCT_PACKAGES += helloworld \
    busybox \
    hellojavajar \
    FirstSystemApp
```

## 三、编译

执行以下命令编译：

```shell
source build/envsetup.sh
lunch rice14-eng
m
```

编译完成后启动模拟器，在模拟器中如果应用列表存在`FirstSystemApp`应用，则表示操作成功。

## 四、系统APP添加自动定义库

其实，我们在开发系统APP需要的大多依赖库在AOSP源码中已经存在，我们只需要在`Android.bp`文件中添加依赖即可。相关目录如下：

- `prebuilts/tools/common/m2`
- `prebuilts/sdk/current/androidx`

但如果系统中不存在我们需要的依赖库，我们可以手动添加三方库。接下来我们添加`lottie`库，在`FirstSystemApp`目录下新建一个`liblottie`目录，然后把下载的lottie依赖包添加到目录下，`lottie`的下载地址为<https://repo1.maven.org/maven2/com/airbnb/android/lottie/5.2.0/lottie-5.2.0.aar>。将lottie添加到本地后，在该目录新建一个`Android.bp`文件，内容如下：

```shell
android_library_import {
    name: "lib-lottie",
    aars: ["lottie-5.2.0.aar"],
    sdk_version: "current"
}
```

最后，我们在`FistSystemApp`目录下`Android.bp`文件中添加`lottie`依赖，如下

```shell
android_app {
    name: "FirstSystemApp",
    srcs: ["src/**/*.java"],
    resource_dirs: ["res"],
    manifest: "AndroidManifest.xml",
    platform_apis: true,
    sdk_version: "",
    certificate: "platform",
    product_specific: true,

    static_libs: [
        "androidx.appcompat_appcompat",
        "com.google.android.material_material",
        "androidx-constraintlayout_constraintlayout",
        "lib-lottie"
    ]
}
```

再进行编译，编译成功，代表自定义系统APP添加三方库成功。
