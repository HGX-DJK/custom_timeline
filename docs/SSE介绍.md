
# SSE（Server-Sent Events）详解

## 🧠 什么是 SSE？

**SSE（Server-Sent Events）** 是 HTML5 标准中定义的一种通信机制，它允许服务器主动将事件推送给客户端（浏览器）。与传统的 HTTP 请求-响应不同，SSE 是一种**基于 HTTP 的单向流式通信协议**。

## 📌 核心特性

| 特性 | 描述 |
|------|------|
| 通信方式 | 单向：**服务器 → 客户端** |
| 连接类型 | **持久的 HTTP 长连接** |
| 数据格式 | 文本（MIME 类型：`text/event-stream`） |
| 自动重连 | 浏览器自动支持 |
| 底层协议 | HTTP/1.1，兼容 HTTP 代理、缓存、身份验证 |
| 建立方式 | 浏览器端使用 `EventSource` 对象 |

## 🧱 通信流程（架构图）

```text
浏览器（客户端）
└── 发起 HTTP GET 请求（Accept: text/event-stream）
    ↓
服务器端（保持连接）
└── 每当有事件，推送：
    data: xxx
    id: 1
    event: myEvent
    retry: 3000

[连接不断，服务器持续发送，客户端持续接收]
```

## 🧪 示例详解

### 1. 客户端（JS）

```javascript
const source = new EventSource('/events');

source.onmessage = (event) => {
  console.log('默认消息:', event.data);
};

source.addEventListener('update', (event) => {
  console.log('收到 update 事件:', event.data);
});

source.onerror = (err) => {
  console.error('连接异常', err);
};
```

### 2. 服务器端（Node.js 示例）

```js
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  res.write('retry: 5000\n'); // 自动重连时间
  res.write('event: update\n');
  res.write(`data: 你好，客户端\n\n`);

  const interval = setInterval(() => {
    res.write(`data: 当前时间 ${new Date().toISOString()}\n\n`);
  }, 3000);

  req.on('close', () => {
    clearInterval(interval);
    res.end();
  });
});
```

## 🎯 典型应用场景

| 场景 | 描述 |
|------|------|
| 实时通知系统 | 消息推送、任务进度提醒 |
| 实时监控面板 | 日志、CPU、内存、API 状态 |
| 数据大屏 | 股票/气象/新闻流更新 |
| 聊天室（只读方） | 只需服务器广播消息 |
| DevTools/日志监听 | 构建日志实时显示 |

## 📄 SSE 消息格式

```text
event: update
id: 12345
retry: 5000
data: 你好
data: 世界
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `data:` | 消息内容，可多行，客户端拼接为一条 |
| `event:` | 事件名称，配合 JS 的 `addEventListener` 使用 |
| `id:` | 消息 ID，客户端会自动缓存用于断线续传 |
| `retry:` | 告诉浏览器下次重连时间（ms） |

## 🧰 与 WebSocket 对比

| 对比项 | SSE | WebSocket |
|--------|-----|-----------|
| 通信方向 | 单向（服务器 → 客户端） | 双向 |
| 协议 | 基于 HTTP | 使用 ws:// 或 wss://（非 HTTP） |
| 使用复杂度 | 简单（浏览器原生支持） | 需要手动管理消息格式、连接状态 |
| 数据格式 | 纯文本（`text/event-stream`） | 文本或二进制 |
| 自动重连 | 浏览器原生支持 | 需要自行实现 |
| 代理支持 | 很好（HTTP 代理可用） | 差（很多 HTTP 代理不支持） |
| 应用场景 | 实时通知、日志流 | 聊天、游戏、协同操作等交互频繁的场景 |

## 📡 浏览器支持情况

| 浏览器 | 支持情况 |
|--------|----------|
| Chrome | ✅ |
| Firefox | ✅ |
| Safari | ✅ |
| Edge（Chromium） | ✅ |
| Internet Explorer | ❌ 不支持 |
| 移动端浏览器 | ✅（多数现代浏览器） |

## ⚠️ 限制与注意事项

1. **不支持 IE**
2. **只能从服务器推送**
3. **受限于 HTTP 连接数**
4. **不适合传输大量二进制数据**

## 🔐 跨域 & 认证

- 使用 CORS 即可跨域支持：

```text
Access-Control-Allow-Origin: *
Content-Type: text/event-stream
```

- URL Token 示例：

```js
new EventSource(`/events?token=abc123`);
```

## 📦 框架与工具支持

| 技术栈 | 支持方式 |
|--------|-----------|
| Node.js | 原生、Express、NestJS |
| Python | Flask + `flask-sse`，Django Channels |
| Java | Spring WebFlux、Servlet 推送 |
| Go | 原生 `net/http` |
| Rust | Actix、Rocket 支持 SSE |
| Vue/React | 用 `EventSource` 封装 Hook/Composables |
| Nginx | 可代理 SSE，需关闭缓存，配置 `proxy_buffering off;` |

## ⚒️ 总结：SSE 的优势与应用判断

- ✅ 简单、轻量级、易于实现和部署（基于 HTTP）
- ✅ 适合实时监控、系统通知、数据流更新
- ❌ 不适合需要客户端发消息或二进制传输
- ❌ 兼容性上需考虑 IE 或企业内网浏览器情况

## 🖥️ SSE和StdioClientTransport对比

> StdioClientTransport 并不是 SSE（Server-Sent Events）的一种实现或使用方式，两者在概念和用途上完全不同

### 两者本质区别

| 特性    | StdioClientTransport         | SSE（Server-Sent Events）        |
| ----- | ---------------------------- | ------------------------------ |
| 类型    | 一种进程间通信（IPC）机制               | 一种基于 HTTP 的网络通信协议              |
| 通信方式  | 使用标准输入输出（stdin/stdout）       | 使用 HTTP 长连接（text/event-stream）    |
| 应用领域  | 主要用于客户端与子进程（如 LSP、AI 工具）之间通信 | 主要用于浏览器实时接收服务器消息  |
| 协议支持  | 非网络协议，依赖本地进程启动               | 网络协议，基于 HTTP                   |
| 使用场景  | VSCode 插件、AI 客户端对接 CLI 工具    | 浏览器实时通知、日志推送等                |
| 是否跨进程 | ✅（父子进程通信）                    | ❌（是跨网络通信）                      |