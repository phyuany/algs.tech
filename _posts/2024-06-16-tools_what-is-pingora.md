---
title: 一天为用户节省434年握手时间！Rust编写的Pingora凭什么力压Nginx？
date: 2024-06-16 15:30:00 +0800
categories: [工具]
tags: [Pingora]
pin: false
---

## Pingora简介

作为一个对 `Rust` 语言和新兴技术充满兴趣的开发者，我最近了解到一个令人振奋的项目——`Pingora`。

这是 `Cloudflare` 使用 `Rust` 构建的全新 `HTTP` 代理，意在替代Nginx。`Pingora` 每天处理超过1万亿个请求，不仅大幅提升了性能，还为客户带来了许多新功能，同时只需以前基础架构的三分之一CPU和内存资源。本文将为大家介绍 `Pingora` 的设计理念以及它所带来的巨大优势。

## 为什么Cloudflare需要一个新的代理？

随着 `Cloudflare` 的快速扩展，现有的 `Nginx` 代理在处理能力和功能方面出现了局限性。虽然 `Nginx` 多年来运作良好，但在`Cloudflare` 的大规模应用中，仍存在一些难以解决的问题：

1. **架构限制损害性能**：`Nginx` 的 `worker` 架构导致 `CPU` 负载不平衡，进而影响整体性能。

2. **糟糕的连接重用**：`Nginx` 的连接池与单个 `worker` 绑定，导致连接重用率低，影响请求首字节时间（`TTFB`）。

3. **功能扩展困难**：`Nginx` 在添加复杂功能时存在局限性，且其用C语言编写存在内存安全问题。

## 构建自有代理的决策过程

面对这些挑战，`Cloudflare` 评估了三种解决方案：

1. 继续投资 `Nginx`，并进行定制化改造。

2. 迁移到其他第三方代理代码库，如 `Envoy`。

3. 从头开始构建一个内部平台和框架。

最终，`Cloudflare` 决定从头开始构建一个适合其需求的新代理系统——`Pingora`。`Pingora` 的设计不仅解决了 `Nginx` 的架构缺陷，还大大提升了性能和效率。

## Pingora项目的设计决策

为了打造一个高效、安全且易于扩展的代理系统，`Cloudflare` 做出了一些关键设计决策：

1. **选择Rust语言**：`Rust` 提供内存安全特性，同时具备 C 语言的高性能，是构建高效代理系统的理想选择。

2. **自建HTTP库**：为了最大化处理HTTP流量的灵活性， `Cloudflare` 选择构建自己的 `HTTP` 库，而不是使用现成的第三方库。

3. **多线程架构**：采用多线程而非多进程架构，以便更好地共享资源，尤其是连接池。

4. **开发者友好的接口**：设计类似 `Nginx/OpenResty` 的基于“请求生命周期”事件的可编程接口，使开发者可以轻松上手并快速开发新功能。

## Pingora在生产环境中的表现

自 `Pingora` 上线以来，它处理了几乎所有需要与源服务器交互的 `HTTP` 请求，性能数据显著提升：

1. **TTFB显著降低**：`Pingora` 上流量的TTFB中位数减少了5毫秒，第95百分位数减少了80毫秒。

2. **连接重用率大幅提升**：`Pingora` 的连接重用率提高，使每秒新连接数减少了三分之二。

3. **资源消耗大幅降低**：在相同流量负载下，`Pingora`的 CPU 和内存消耗减少了约70%和67%。

自 `Cloudflare` 上线 `Pingora` 以来，它处理了几乎所有需要与源服务器交互的 `HTTP` 请求，性能数据显著提升。

让我们看看 `Cloudflare` 使用 `Pingora` 如何加快客户的流量。

`Pingora` 的总体流量数据显示，`TTFB` 中位数减少了5毫秒，第95个百分位数减少了80毫秒。主要是得益于新架构可以跨所有线程共享连接，从而提高了连接重用率，减少了 `TCP` 和 `TLS` 握手时间。

与旧服务相比，`Pingora` 每秒的新连接数减少到原来的三分之一。对于某个主要客户，连接重用率从87.1%提升到了99.92%，这将新连接减少了160倍。更直观地说，通过切换到 `Pingora`，`Cloudflare` 每天为客户和用户节省了434年的握手时间。

## 更高效、更安全、更强大

`Pingora`不仅性能出众，还在功能扩展和安全性方面表现优异：

1. **功能扩展更灵活**：例如，`Cloudflare` 能够轻松添加`HTTP/2` 上游支持，并向客户提供 `gRPC` 服务。

2. **更高效的资源使用**：`Rust` 代码和多线程模型使得资源使用更高效，减少了连接创建和数据共享的成本。

3. **内存安全**：`Rust` 的内存安全语义大大减少了内存错误的可能性，确保服务稳定运行。

## 单负载均衡器的示例代码

为了更好地展示 `Pingora` 的强大功能，下面提供一个使用Pingora实现简单负载均衡器的示例代码：

```rust
use async_trait::async_trait;
use pingora::prelude::*;
use std::sync::Arc;

fn main() {
    // 创建一个服务器实例，参数为None表示使用默认配置
    let mut my_server = Server::new(None).unwrap();
    // 初始化服务器
    my_server.bootstrap();
    // 创建一个负载均衡器，包含两个上游服务器
    let upstreams = LoadBalancer::try_from_iter(["192.168.9.34:80", "10.0.0.9:80"]).unwrap();
    // 创建一个HTTP代理服务，并传入服务器配置和负载均衡器
    let mut lb = http_proxy_service(&my_server.configuration, LB(Arc::new(upstreams)));
    // 添加一个TCP监听地址
    lb.add_tcp("0.0.0.0:6188");
    // 将服务添加到服务器中
    my_server.add_service(lb);
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
        let peer: Box<HttpPeer> = Box::new(HttpPeer::new(upstream, false, "example.com".to_string()));
        Ok(peer)
    }

    // 在上游请求发送前，插入Host头部
    async fn upstream_request_filter(
        &self,
        _session: &mut Session,
        upstream_request: &mut RequestHeader,
        _ctx: &mut Self::CTX,
    ) -> Result<()> {
        // 将Host头部设置为example.com
        upstream_request
            .insert_header("Host", "example.com")
            .unwrap();
        Ok(())
    }
}
```

## 结语

Pingora的发布标志着代理技术领域的一次重大飞跃。我相信pingora也将成为rust一个有意义项目。如果你对互联网新兴技术、rust语言 等感兴趣，欢迎关注我的后续文章和技术分享！

> 本文原创首发自公众号：极客开发者，禁止转载！
