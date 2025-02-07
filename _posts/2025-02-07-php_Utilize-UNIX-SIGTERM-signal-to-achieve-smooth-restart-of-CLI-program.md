---
title: 利用 UNIX SIGTERM 信号实现 CLI 程序平滑重启
date: 2025-02-07 09:54:00 +0800
categories: [PHP]
tags: []
pin: false
---

## 一、引言

当我们开发后台任务或服务时，如何优雅地终止程序是一个绕不开的话题。特别是在频繁更新代码的场景下，程序突然中断可能导致任务中断或数据不一致的问题。

本文以简单的 `PHP CLI` 为示例，将从 **SIGTERM 信号** 的基本概念讲起，逐步深入，带你掌握如何在 **PHP CLI 程序** 中实现优雅的平滑重启。

## 二、SIGTERM 信号的基本概念

### 2.1 什么是 SIGTERM 信号？

**SIGTERM** 是 Unix 和类 Unix 系统中用于终止进程的信号之一。它的特别之处在于，进程在收到该信号时，有机会决定如何应对：

1. **直接退出**：如果程序没有捕获或处理 SIGTERM，操作系统会按照默认行为，直接终止进程。
2. **优雅退出**：程序可以通过捕获 SIGTERM，执行一些必要的清理逻辑，例如保存状态、释放资源等，然后再退出。

与其他信号相比：

- **SIGINT**（通常由用户按下 `Ctrl + C` 触发）用于交互式中断。
- **SIGKILL** 则无法被捕获，直接强制终止进程。

### 2.2 SIGTERM 在应用程序中的意义

在应用程序中，优雅地终止进程非常重要，尤其是在以下场景中：

1. **后台任务处理**：防止任务中断，保证关键操作的完整性。
2. **服务热更新**：当部署新版本时，旧实例需要平稳退出，避免中途终止正在处理的请求。
3. **资源释放**：如关闭数据库连接、文件句柄等，防止资源泄漏。

现代工具如 **Kubernetes** 和 **supervisord**，在停止容器或重启服务时，都会发送 SIGTERM 信号。这使得进程有时间完成未完成的任务，再优雅地退出。

## 三、如何在 PHP CLI 程序中实现优雅重启？

为了让 PHP CLI 程序能够捕获 SIGTERM 信号，并在收到信号时优雅退出，可以使用 PHP 提供的 **`pcntl_signal`** 和 **`pcntl_async_signals`** 函数。

### 3.1 基本实现：捕获 SIGTERM 信号

以下代码展示了如何捕获 SIGTERM 信号并进行清理：

```php
<?php

declare(ticks=1); // 启用信号检测

// 注册信号处理器
pcntl_signal(SIGTERM, function () {
    echo "Received SIGTERM, shutting down gracefully...\n";
    // 在这里添加清理逻辑，例如保存状态、释放资源等
    exit(0); // 确保程序优雅退出
});

while (true) {
    echo "Running a task...\n";
    sleep(1); // 模拟任务处理
}
```

**运行效果：**

- 当程序运行时，如果发送 `SIGTERM` 信号（例如使用 `kill` 命令），程序会捕获信号并执行清理逻辑后退出。

### 3.2 实现优雅重启逻辑

如果你的程序是一个后台任务处理器，需要确保当前任务在退出前执行完毕，可以使用以下方式：

```php
<?php

declare(ticks=1);

// 注册信号处理器
$shouldStop = false;
pcntl_signal(SIGTERM, function () use (&$shouldStop) {
    echo "Received SIGTERM, waiting for the current task to complete...\n";
    $shouldStop = true;
});

while (true) {
    // 模拟任务处理
    echo "Processing a task...\n";
    sleep(5); // 每个任务需要 5 秒完成

    // 检查是否收到停止信号
    if ($shouldStop) {
        echo "Task completed, shutting down gracefully...\n";
        break;
    }
}
```

### 3.3 优雅重启逻辑优化

以下是优化的示例代码：

```php
<?php

pcntl_async_signals(true);

$shouldStop = false;

// 注册信号处理器, 处理 SIGTERM 和 SIGINT 信号
pcntl_signal(SIGTERM, function () use (&$shouldStop) {
    echo "Received SIGTERM, waiting for the current task to complete...\n";
    $shouldStop = true;
});
pcntl_signal(SIGINT, function () use (&$shouldStop) {
    echo "Received SIGINT (Ctrl + C), waiting for the current task to complete...\n";
    $shouldStop = true;
});

// 处理任务的方法
function processTask()
{
    // 模拟任务处理
    echo "Processing a task...\n";
    
    // 每隔1秒，输出一次信息，一共输出5次
    for ($i = 1; $i <= 5; $i++) {
        echo "Task progress: $i/5\n";
        sleep(1);
    }

    echo "Task completed.\n";
}

while (true) {
    // 模拟任务处理
    processTask();

    // 检查是否收到停止信号
    if ($shouldStop) {
        echo "Task completed, shutting down gracefully...\n";
        break;
    }
}
```

`pcntl_async_signals(true)` 是 PHP 7.0 之后引入的功能，用于启用异步信号处理。它使得程序能够在后台自动响应信号，而不需要依赖 `declare(ticks=N)`，程序不需要显式地检查信号，信号会自动处理，提高性能。

另外我们增加了 `SIGINT` 信号的处理逻辑，以支持用户手动终止（用户按 `Ctrl + C`）程序的场景。

## 四、SIGTERM 实际应用场景

程序会等待当前任务处理完成后再退出，避免中途中断任务。

### 4.1 Kubernetes 中的优雅终止

在 `Kubernetes` 中，当一个 `Pod` 被删除时，`Kubernetes` 会发送 `SIGTERM` 信号给容器的主进程（通常是你的应用）。通过捕获 `SIGTERM`，可以让服务完成当前请求后再退出。

### 4.2 supervisord 中的优雅重启

在使用 `supervisord` 管理进程时，执行 `supervisorctl restart` 或 `stop` 时，会发送 `SIGTERM` 信号。如果未捕获该信号，程序会被立即终止，可能导致任务中断。

### 4.3 最佳实践建议

1. **保持清理逻辑轻量化**: 捕获 `SIGTERM` 后的逻辑应尽量简单，比如标记状态、保存数据，而不是执行复杂操作;
2. **避免长时间阻塞**: 设置合理的超时时间，确保程序不会因为清理逻辑过慢而无法退出;
3. **考虑用户手动停止**: 在用户使用 `Ctrl + C` 停止程序时，程序会收到 `SIGINT` 信号，可以通过处理 `SIGINT` 信号的方式，执行一些必要的清理逻辑再退出。在实际场景中，可以根据情况进行选择处理 `SIGINT` 信号。

## 五、总结

掌握 `SIGTERM` 信号的处理，不仅能帮助你编写更可靠的程序，还能提升服务的稳定性和可维护性。希望这篇教程能帮你轻松掌握程序优雅重启的方法！
