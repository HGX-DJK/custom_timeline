
# 🧠 什么是 IPC（Inter-Process Communication）？

**IPC（进程间通信）** 是指操作系统提供的一种机制，允许不同的进程之间交换数据、消息或信号。

> 一个进程是操作系统资源分配的基本单位，不同进程通常拥有各自独立的地址空间。为了协同工作，它们需要通过 IPC 来交换信息。

---

## 📦 IPC 的常见类型（按通信方式分类）

| 类型 | 是否跨主机 | 是否需要共享内存 | 是否支持同步 | 性能 | 描述 |
|------|------------|------------------|--------------|--------|------|
| 管道（Pipe） | 否 | 否 | 否 | 中 | 单向通信，父子进程之间 |
| 有名管道（FIFO） | 否 | 否 | 否 | 中 | 可以在非亲缘进程间使用 |
| 消息队列（Message Queue） | 否 | 否 | 是 | 中 | 操作系统内核维护队列 |
| 信号（Signal） | 否 | 否 | 否 | 高 | 发送特定事件信号 |
| 共享内存（Shared Memory） | 否 | 是 | 否 | 高 | 多个进程共享一块内存区域 |
| 信号量（Semaphore） | 否 | 否 | 是 | 高 | 用于进程同步/互斥 |
| 套接字（Socket） | ✅ | 否 | 否 | 低（跨网络） | 支持网络和本地通信 |
| 标准输入输出（Stdio） | 否 | 否 | 否 | 高 | 子进程通过 stdin/stdout 与主进程通信 |

---

## 🧱 常见 IPC 方式详解

### 1. 管道（Pipe）

- 匿名管道：仅限于父子进程通信。
- 特性：半双工，一端写，一端读。
- 示例（Linux shell）：
  
  ```bash
  ls | grep txt
  ```

### 2. 有名管道（FIFO）

- 创建文件名用于通信，允许非父子进程间使用。
- 示例：

  ```bash
  mkfifo /tmp/myfifo
  echo "Hello" > /tmp/myfifo
  cat /tmp/myfifo
  ```

### 3. 消息队列（Message Queue）

- 操作系统提供队列，进程可向其中发送或接收消息。
- 示例函数：`msgsnd()` / `msgrcv()`（Unix）

### 4. 信号（Signal）

- 用于进程通知，如 `SIGINT`, `SIGKILL`, `SIGUSR1`。
- 示例（发送信号）：

  ```bash
  kill -USR1 <pid>
  ```

### 5. 共享内存（Shared Memory）

- 多个进程访问一块共享内存，速度最快。
- 需配合信号量实现同步。

### 6. 信号量（Semaphore）

- 用于控制对共享资源的访问，实现同步与互斥。

### 7. 套接字（Socket）

- 支持本地通信，也支持网络间通信。
- 常用于客户端/服务端架构。

### 8. 标准输入输出（Stdio）

- 父进程通过 stdin 写入，stdout 读取子进程数据。
- 常用于 JSON-RPC、LSP、AI 工具。

---

## 🔄 常见对比表

| IPC方式 | 跨平台 | 同步控制 | 适用场景 | 速度 |
|---------|--------|------------|-----------|--------|
| Pipe | ✅ | ❌ | 父子进程通信 | 中 |
| FIFO | ✅ | ❌ | 任意进程通信 | 中 |
| Message Queue | ✅ | ✅ | 多任务消息传递 | 中 |
| Shared Memory | ✅ | ❌ | 大数据高效传输 | ✅ 高 |
| Semaphore | ✅ | ✅ | 同步与互斥 | ✅ 高 |
| Socket | ✅ | ✅ | 网络通信、客户端/服务端模型 | ❌ 慢 |
| Stdio | ✅ | ❌ | 启动本地服务、脚本通信 | ✅ 快 |

---

## 💡 实用示例：Node.js + Python 使用 Stdio 通信

### Node.js 主程序

```ts
import { spawn } from "child_process";

const py = spawn("python", ["my_script.py"]);
py.stdin.write("hello\n");

py.stdout.on("data", (data) => {
  console.log("Python 返回:", data.toString());
});
```

### Python 子程序

```python
import sys

for line in sys.stdin:
    sys.stdout.write(f"接收到：{line}")
    sys.stdout.flush()
```

---

## 🚀 应用场景总结

| 场景 | 推荐 IPC 方式 |
|------|----------------|
| 编辑器插件（如 VSCode LSP） | `stdio` 或 `socket` |
| 网络服务（浏览器通信） | `WebSocket`, `HTTP`, `socket` |
| 容器间通信 | `socket` 或 `message queue` |
| 高速大数据通信 | `shared memory` + `semaphore` |
| 简单脚本交互 | `pipe` 或 `stdio` |

---

## ✅ 总结

- IPC 是实现多进程协作的核心机制。
- 不同 IPC 方式适配不同场景：性能、同步、跨主机是主要考量维度。
- 在前后端协作、服务编排、本地 AI 工具调用中，IPC 是不可或缺的基础设施。
