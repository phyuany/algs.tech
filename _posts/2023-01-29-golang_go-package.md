---
title: golang语言中包的使用
date: 2023-01-29 21:29:00 +0800
categories: [golang]
tags: []
pin: false
---

> 撰写时间：2021-07-05，整理时间：2023-01-29

在 golang 中，所有源文件都属于一个包，golang 的包具有以下特性

- 包可以被其他包引用
- 每个 golang 程序只有一个 main 包
- 包的主要用途是提高代码的可复用性

本节，我们将介绍包的相关概念以及使用方法，如果你还没有阅读过前面的文章，可以通过以下链接进行阅读

[golang 的安装](https://blog.jkdev.cn/index.php/archives/332/)

## 一、GOPATH 环境变量

### 1.1 什么是 GOPATH？

golang 包的使用依赖一个重要的环境变量：`GOPATH`，无论是自己的代码还是第三方代码，都以`GOPATH`作为工作目录和一套完整的工程目录规则。GOPATH 目录约定有三个子目录，主要用于

- **src**：存放各种包源代码，src 里的一个目录就是一个包
- **pkg**：存放编译源代码后的类库
- **bin**：存放编译源代码后的可执行程序

`GOPATH`允许多个目录，设置多个目录时，需要使用环境变量分隔符来区分。Unix/Linux 的环境变量分隔符是英文输入法下的冒号（:），windows 的环境变量分隔符是英文分号（;）。

使用`GOPATH`时，golang 会在以下目录中搜索包

- GOROOT/src：该目录保存了 Go 标准库里代码。
- GOPATH/src：该目录保存了应用自身的代码和第三方依赖的代码。

### 1.2 设置 GOPATH 环境变量

在 Unix/Linux 中，使用`sudo vim /etc/profile`打开系统 profile 文件，并追加以下内容设置`GOPATH`环境变量，同时设置`GO111MODULE`为关闭状态，`GO111MODULE`环境变量我们在文章中后面部分说明

```shell
export GOPATH=/Users/pan/go # 把自己的golang项目目录作为GOPATH的变量
export GO111MODULE=off # 关闭golang 11.1 版本以后的模块功能
```

使用`source /etc/profile`命令重新加载环境变量

### 1.3. 包的使用

- 引入系统包

我们可以通过`package`关键字定义一个包，使用`import`引入其他包，如果需要引入多个包，可以将各个包声明在括号内

```golang
package main

import (
    // 引入系统的包
    "fmt"
)

func main() {
    // 调用系统包的函数
    fmt.Println("Hello World")
}
```

在以上代码中，我们通过`main`关键字定义了 main 包，主函数 main 必须位于 main 包下。程序中引入了`fmt`包，该包由 golang 本身提供。

- 引入自定义包
我们在 GOPATH/src 目录下创建两个文件

文件 1，库文件位于`jkdev.cn/test/mypkg/lib.go`，添加以下代码

```golang
package mypkg

import "fmt"

func TestFunc() {
    fmt.Println("call testFunc")
}
```

文件 2，主文件`jkdev.cn/test/main.go`，添加以下代码

```golang
package main

// 引入自定义包
import "jkdev.cn/test/mypkg"

func main() {
    // 调用自定义包的函数
    mypkg.TestFunc()
}
```

自定义的包由我们自己定义，但要注意的是，一个目录对应一个包，同时，包名应当和目录名一致。引入包时，路径相对应 GOPATH/src，我们可以根据自己的程序的功能模块划分不同的包。

- 包的作用域

**包内**
在同一个包内定义的函数、变量、常量、结构体，可以被包内的所有其他代码任意访问，它们属于包内公开。

**包外**
如果函数、变量、常量、结构体位于不同的包下，如果将它们的首字母使用大写标识，表示它们是公开访问的。对于结构体字段。如果想要在包外进行访问，还要让结构体字段变量名使用首字母大写。

## 二、 编译

### 2.1 概述

golang 中`go build` 命令主要用于编译代码。在包的编译过程中，若有必要，会同时编译与之相关联的包。

假设我们的设置 GOPATH 是`/Users/pan/go`，我们在 GOPATH 之下的 src 创建创建代码 test 目录，并添加`lib.go`和`main.go`两个文件：

此时，源文件相对于 GOPATH 的路径如下。接下来我们对这两个 go 源文件进行编译

```shell
go  GOPATH目录
├─src                  源码目录
│  ├─test              项目目录
│     ├─lib.go         公共函数文件
│     └─main.go        主函数文件
```

lib.go 代码如下：

```golang
package main

import "fmt"

func testFunc() {
    fmt.Println("call testFunc")
}
```

main.go 代码如下：

```go
package main

func main() {
    testFunc()
}
```

示例中的两个源文件都属于`main`包，所以在 main.go 文件中能正常引入`lib.go`文件。

### 2.2 go build

无参数编译不依赖`GOPATH`，需要在源码目录下进行编译

```shell
# 进入源码目录
cd ~/go/src/test
# 编译代码
go build
```

我们还可以`-o`指定编译后输出的可执行文件的文件名，如下

```shell
go build -o main
```

### 2.3 go build 文件列表

使用“go build+文件列表”编译时，文件列表中的每个文件必须是同一个包的 Go 源码

```shell
# 进入源码目录
cd ~/go/src/test
# 编译
go build lib.go main.go
```

### 2.4 go build 包

包的路径相对于`GOPATH`，“go build+包”编译时，源文件需要存放在`GOPATH/src`下的路径，但是我们可以在任意有读写权限的目录执行编译命令，如下

```shell
# test 相对于GOPATH/src
go build test
```

### 2.5 go install

与`go build`具有相似功能的是`go install`命令，`go install`用来生成库和工具并存放于 GOPATH 中。一是编译包文件（无 main 包），将编译后的包文件放到 pkg 目录下（GOPATH/pkg）。二是编译生成可执行文件（有 main 包），将可执行文件放到 bin 目录（GOPATH/bin）。

### 2.6 go build 和 go install 对比

- 相同点
都能生成可执行文件

- 不同点
go build 不能生成包文件, go install 可以生成包文件；go build 生成可执行文件在当前目录下， go install 生成可执行文件在 bin 目录下（$GOPATH/bin）。

## 三、 Go 模块

golang 在 1.11 版本引入和 Go Modules，我们还可以叫做 Go 模块，模块依赖于上文提到的`GO111MODULE`环境变量。它是 golang 官方提供的包管理工具，通过 Go Modules，我们可以不必将项目放在 GOPATH 上。

### 3.1 GO111MODULE 环境变量

用环境变量 GO111MODULE 设置开启或者关闭模块支持，它有三个可选值：off、on、auto，默认是 auto。

- `GO111MODULE=off`：无模块支持，go 会从 GOPATH 寻找包。
- `GO111MODULE=on`：模块支持，根据`go.mod`文件下载依赖
- `GO111MODULE=auto`：在$GOPATH/src 外面且项目根目录有`go.mod`文件时，自动开启模块支持

在使用模块的时候，GOPATH 是无意义的，但 golang 还是会把下载的依赖存储于 GOPATH 之下。

### 3.2 使用 go 模块

使用 Go Modules 时，我们先将`GO111MODULE`设置为 on 或者 auto，Go Modules 和 GOPATH 不同，不需要包含固定的三个字目录。一个空的源代码目录，只要初始化好`go.mod`文件，就可以定义一个 Module。我们可以使用以下命令创建一个新的 Module

```shell
# 创建一个空的工作目录
mkdir -p ~/Work/go/test
# 进入工作目录
cd ~/Work/go/test
# 初始化一个module
go mod init testmodule
```

我们可以看到自动生成的`go.mod`文件的内容如下

```go
module testmodule

go 1.16
```

- 下载依赖

Go Modules 会为我们进行包管理，并自动更新 go.mod 文件，如果需要引入新的依赖，我们只需要在终端执行`go get 包名`即可引入第三方包，如

```shell
go get github.com/satori/go.uuid
```

执行命令后，Go Modules 会从我们在安装 golang 时设置的`GOPROXY`镜像仓库地址去下载对应的包，安装完成之后我们看到`go.mod`已经发生变化，如下

```go
module testmodule

go 1.16

require github.com/satori/go.uuid v1.2.0 // indirect
```

我们也可以先编辑好`go.mod`文件，然后在终端使用`go mod download`，golang 会自动帮我们下载依赖包到本地。如果我们需要引入多个包，将所有包名卸载括号内，如下代码

```golang
module testmodule

go 1.16

require (
    github.com/satori/go.uuid v1.2.0 // indirect
    gorm.io/gorm v1.21.11 // indirect
)
```

- 更新依赖关系

使用`go mod tidy`更新依赖关系，该命令拉取缺少的模块，移除不用的模块。Go Modules 可以轻易地进行一个包的依赖管理和版本控制，`go build`和`go install`将自动使用 go.mod 中的依赖关系，减少了 GOPATH 管理时的复杂性。

> 本文介绍了 Go 的包的基本管理方法，后面，我们会循序渐进地介绍 golang 的基础知识。如果喜欢，欢迎订阅！
