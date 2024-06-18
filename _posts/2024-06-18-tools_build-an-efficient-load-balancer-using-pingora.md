---
title: Pingora高效负载均衡器的构建指南
date: 2024-06-18 15:30:00 +0800
categories: [工具]
tags: [Pingora]
pin: false
---

## 一、什么是 Pingora？

`Pingora` 是一个高性能的 `Rust` 库，用于构建负载均衡器和代理服务器。它提供了丰富的功能和高度的扩展性，适用于各种网络应用场景。今天，我带大家一步一步使用 `Rust` 生态中的 `Pingora` 构建一个基础的负载均衡器。

![24061901](/img/tools/24061901.jpg)

## 二、实现过程

### 2.1 准备工作

首先，我们需要一个 Rust 项目，并添加必要的依赖项。在项目根目录下的 `Cargo.toml` 文件中添加以下内容：

```toml
[package]
name = "load_balancer"
version = "0.1.0"
edition = "2021"

[dependencies]
async-trait = "0.1"
pingora = { version = "0.1", features = ["lb"] }
```

### 2.2 代码实现

接下来，我们在 `src/main.rs` 中编写负载均衡器的实现代码。以下是完整的代码示例：

```rust
use async_trait::async_trait;
use pingora::{prelude::*, services::Service};
use std::sync::Arc;

fn main() {
    // 创建一个服务器实例，参数为None表示使用默认配置
    let mut my_server = Server::new(None).unwrap();
    // 初始化服务器
    my_server.bootstrap();
    // 创建一个负载均衡器，包含两个上游服务器
    let upstreams = LoadBalancer::try_from_iter(["10.0.0.1:8080", "10.0.0.2:8080"]).unwrap();
    // 创建一个HTTP代理服务，并传入服务器配置和负载均衡器，这里的负载均衡器实现了ProxyHttp trait
    let mut lb_service: pingora::services::listening::Service<pingora::proxy::HttpProxy<LB>> =
        http_proxy_service(&my_server.configuration, LB(Arc::new(upstreams)));
    
    // 添加一个TCP监听地址，监听80端口
    lb_service.add_tcp("0.0.0.0:80");

    // 在项目目录下新增一个 keys 目录，对应证书文件放在该目录下。最后添加一个TLS监听地址，监听443端口
    let cert_path = format!("{}/keys/example.com.crt", env!("CARGO_MANIFEST_DIR"));
    let key_path = format!("{}/keys/example.com.key", env!("CARGO_MANIFEST_DIR"));
    let mut tls_settings =
        pingora::listeners::TlsSettings::intermediate(&cert_path, &key_path).unwrap();
    tls_settings.enable_h2();
    lb_service.add_tls_with_settings("0.0.0.0:443", None, tls_settings);

    // 定义服务列表，可以添加多个服务，这个示例只有一个负载均衡服务。将服务列表添加到服务器中
    let services: Vec<Box<dyn Service>> = vec![Box::new(lb_service)];
    my_server.add_services(services);

    // 运行服务器，进入事件循环
    my_server.run_forever();
}

// 定义一个包含负载均衡器的结构体LB，用于包装Arc指针以实现多线程共享。
pub struct LB(Arc<LoadBalancer<RoundRobin>>);

// 使用#[async_trait]宏，异步实现ProxyHttp trait。
#[async_trait]
impl ProxyHttp for LB {
    /// 定义上下文类型，这里使用空元组。对于这个小例子，我们不需要上下文存储
    type CTX = ();
    // 创建新的上下文实例，这里返回空元组
    fn new_ctx(&self) -> () {
        ()
    }
    // 选择上游服务器并创建HTTP对等体
    async fn upstream_peer(&self, _session: &mut Session, _ctx: &mut ()) -> Result<Box<HttpPeer>> {
        // 使用轮询算法选择上游服务器
        let upstream = self
            .0
            .select(b"", 256) // 对于轮询，哈希不重要
            .unwrap();
        println!("上游对等体是：{upstream:?}");
        // 创建一个新的HTTP对等体，设置SNI为example.com
        let peer: Box<HttpPeer> =
            Box::new(HttpPeer::new(upstream, false, "example.com".to_string()));
        Ok(peer)
    }

    // 在上游请求发送前，执行一些额外操作，例如将某些参数插入请求头。这里的示例是插入Host头部
    async fn upstream_request_filter(
        &self,
        _session: &mut Session,
        upstream_request: &mut RequestHeader,
        _ctx: &mut Self::CTX,
    ) -> Result<()> {
        // 将Host头部设置为example.com，当然，在现实需求中，这一步可能是多余的
        upstream_request
            .insert_header("Host", "example.com")
            .unwrap();
        Ok(())
    }
}
```

在以上的代码中，我们首先创建了一个服务器实例，然后初始化服务器，创建负载均衡器结构体 `LB` 并实现了 `ProxyHttp trait`。任何实现了 `ProxyHttp trait` 的对象本质上定义了代理中如何处理请求。`ProxyHttp trait` 中必须需要的方法是 `upstream_peer()`，它返回请求应该被代理到的地址。`LB` 结构体对象监听了80端口和443端口，这两个端口分别用于HTTP和HTTPS请求。

## 三、代码优化

以上代码实现了需求，但是代码存在一些优化空间。`Pingora` 提供了一些有用的功能，只需调用对应的代码就可以启用。下面我们对以上代码进行优化：

### 3.1 对等体健康检查

为了使我们的负载均衡器更可靠，我们希望添加健康检查功能到我们的上游对等体。这样，如果有一个对等体已经出现异常，就可以快速停止将流量路由到该对等体。我们先添加包含已宕机的对等体，如下代码

```rust
fn main() {
    // ...
    let upstreams =
        LoadBalancer::try_from_iter(["10.0.0.1:8080", "10.0.0.2:8080", "10.0.0.3:8080"]).unwrap();
    // ...
}
```

现在如果我们再次运行我们的负载均衡器 `cargo run`，并用以下命令测试它：

```shell
curl http://127.0.0.1 -svo /dev/null
```

我们发现会出现 `502: Bad Gateway` 的失败情况，这是因为我们的对等体选择严格遵循我们给出的 `RoundRobin` 选择模式，而没有考虑该对等体是否健康。我们可以通过引入一个健康检查的功能来解决这个问题，进而排除掉不健康对等体。

```rust
fn main() {
    // ...
    // 创建一个负载均衡器的上游对等体列表
    let mut upstreams = LoadBalancer::try_from_iter(["10.0.0.1:8080", "10.0.0.2:8080"]).unwrap();

    // 健康检查
    let hc = TcpHealthCheck::new();
    upstreams.set_health_check(hc);
    upstreams.health_check_frequency = Some(std::time::Duration::from_secs(1));
    let background = background_service("health check", upstreams);
    let upstreams = background.task();

    // 创建一个HTTP代理服务，并传入服务器配置和负载均衡器
    let mut lb_service: pingora::services::listening::Service<pingora::proxy::HttpProxy<LB>> =
        http_proxy_service(&my_server.configuration, LB(upstreams));
    // ...
}
```

### 3.2 接收命令行参数

在创建 `pingora` 服务时，需要传入一个 `Opt::default()` 参数，`pingora` 将会捕获我们运行的命令行参数，并使用这些参数来配置 `pingora` 服务。代码变更如下

```rust
fn main() {
    // ...
    let mut my_server = Server::new(Some(Opt::default())).unwrap();
    // ...
}
```

通过这个更改，`pingora` 服务将能够接收到命令行参数，并使用这些参数来配置自己的运行环境。我们可以通过以下命令来看 `pingora` 负载均衡器的参数说明

```shell
cargo run -- -h
```

通过以上命令，我们可以了解到 `pingora` 相关参数提供的功能，后续可以为我们的服务器实现更多的功能。

## 四、部署

### 4.1 后台运行

通过传递 `-d` 或者 `--daemon` 参数，可以将 `pingora` 运行在后台。如果要优雅的停止 `pingora`，可以使用 `pkill` 命令并且传递 `SIGTERM` 信号，那么在关闭的过程中，服务将停止接收新的请求，但是仍然会处理完当前请求再退出。命令如下

```shell
# 后台运行
cargo run -- -d
# 优雅的停止
pkill -SIGTERM load_balancer
```

### 4.2 配置

Pingora 配置文件可以定义Pingora 如何运行，以下定义了 Pingora 的版本、线程数、pid文件、错误日志文件、升级套接字文件的配置，文件名称命名为`conf.yaml`

```shell
---
version: 1
threads: 2
pid_file: /tmp/load_balancer.pid
error_log: /tmp/load_balancer_err.log
upgrade_sock: /tmp/load_balancer.sock
```

加载配置文件运行如下：

```shell
# 设置日志级别
RUST_LOG=INFO
# 启用
cargo run -- -c conf.yaml -d
```

### 4.3 优雅地升级

假设我们更改了负载均衡器的代码并重新编译了二进制文件。现在我们希望将正在后台运行的服务升级到这个新版本。如果我们简单地停止旧服务，然后启动新服务，那么在中间到达的一些请求可能会丢失。幸运的是，Pingora 提供了一种优雅的方式来升级服务。

首先，我们通过`SIGQUIT`停止正在运行的服务，然后使用`-u`或者`--upgrade`参数来启动全新的程序，如下命令

```shell
pkill -SIGQUIT load_balancer && RUST_LOG=INFO cargo run -- -c conf.yaml -d -u
```

在升级过程中，Pingora 将会自动将请求路由到新的服务，而不会丢失任何请求。从客户端的角度来看，用户感觉不到任何变化。

## 五、总结

到此为止，我们已经拥有了一个功能完备的负载均衡器。通过这个简单的示例，相信大家已经对 Pingora 有了一个初步的了解。

不过，这是一个非常基础的负载均衡器。在实际应用中，负载均衡器的配置和功能可能会更加复杂，我们还需要根据实际需求来进行扩展和优化。

在后续，我也会分享一些关于 Pingora 以及新兴热门技术的更多内容，欢迎继续关注！

> 本文完整示例代码：<https://github.com/phyuany/simple-pingora-reverse-proxy>
