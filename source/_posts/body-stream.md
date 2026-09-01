---
title: HTTP 流式响应：fetch、ReadableStream 与 SSE
date: 2025-12-29 20:02:01
categories:
  - 技术
tags:
  - 网络
---

`fetch()` 收到响应头后返回 `Response`，此时响应体可能仍在传输。`response.body` 将响应体暴露为 `ReadableStream<Uint8Array>`，供 JavaScript 增量读取。

<!-- more -->

## 分层

```text
TCP / QUIC
  ↓
HTTP/1.1 chunked encoding / HTTP/2 DATA frame / HTTP/3 stream
  ↓
SSE / NDJSON / 长度前缀等消息分帧
  ↓
fetch + ReadableStream / EventSource / 其他客户端 API
```

SSE / NDJSON / 长度前缀负责消息分帧；`fetch + ReadableStream` / `EventSource` / OkHttp / curl 负责读取。`EventSource` 固定处理 SSE，`fetch()` 可读取任意分帧格式。

## Fetch API

- `await response.json()`：读取完整 Body，再解析 JSON
- `await response.text()`：读取完整 Body，再解码文本
- `response.body`：获得 `ReadableStream<Uint8Array>`，增量读取字节

```ts
const response = await fetch("/stream");
if (!response.ok) throw new Error(`HTTP ${response.status}`);
if (!response.body) throw new Error("ReadableStream is unavailable");

const reader = response.body.getReader();
const decoder = new TextDecoder("utf-8");

for (;;) {
  const { value, done } = await reader.read();
  if (done) break;

  const text = decoder.decode(value, { stream: true });
  console.log(text);
}

const tail = decoder.decode();
if (tail) console.log(tail);
reader.releaseLock();
```

`read()` 返回当前可用的字节 chunk，大小由网络栈、HTTP 实现与缓冲状态共同决定。

## chunk 与消息边界

一次服务端 `write()`、一个 HTTP DATA frame、一次客户端 `read()`，三者通常不会对齐。一个 chunk 可能包含：

- 半个 UTF-8 字符
- 半个 JSON 对象
- 多条完整消息
- 一条消息的结尾和下一条消息的开头

客户端必须维护跨 chunk 缓冲：

```text
读取字节 → 增量解码 → 按协议分帧 → 解析消息 → 更新业务状态
```

UTF-8 是变长编码。`decoder.decode(value, { stream: true })` 会保留末尾未完成的字节序列，等待下一个 chunk；流结束后调用一次无参 `decode()` 冲刷解码器。也可以使用：

```ts
const textStream = response.body.pipeThrough(new TextDecoderStream());
```

## SSE 与 fetch 的关系

[SSE](https://www.yigegongjiang.com/2024/sse/) 规定事件格式与空行分帧；`fetch()` 负责发起 HTTP 请求，并把响应体暴露为 `ReadableStream`。

同一份 SSE 响应有两种读取方式：

- `EventSource('/events')`：浏览器解析事件，并提供自动重连与 `Last-Event-ID`
- `fetch('/events')`：应用读取字节，自行实现 SSE 解析、终止判断、重连与断点续传

需要 POST、请求体或 `Authorization` Header 时，可使用 `fetch + ReadableStream + SSE framing`。MIME 与线格式仍为 `text/event-stream`；解析、终止判断、重连和断点续传由应用处理。

## 其他分帧格式

### NDJSON

```text
{"type":"progress","percent":40}\n
{"type":"done"}\n
```

- 每行一个完整 JSON
- 按 `\n` 切行后直接 `JSON.parse`
- `application/x-ndjson`
- JSON 文本内部的换行必须转义

NDJSON 解析器应接受 `\n` 与 `\r\n`。标准 JSON 序列化会把字符串内的换行编码为 `\n`，不会破坏行边界。

### 长度前缀

```text
[length][payload][length][payload]
```

长度前缀适合二进制内容或不可控 payload。客户端需要按字节维护状态机，实现成本高于 SSE 与 NDJSON。

## NDJSON 读取器

```ts
type StreamMessage =
  | { type: "progress"; percent: number }
  | { type: "done" }
  | { type: "error"; message: string };

export async function readProgress(
  input: { taskId: string },
  onMessage: (message: StreamMessage) => void,
  signal?: AbortSignal,
) {
  const response = await fetch("/api/progress", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(input),
    signal,
  });

  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  if (!response.body) throw new Error("ReadableStream is unavailable");

  const reader = response.body.getReader();
  const decoder = new TextDecoder("utf-8");
  let buffer = "";

  const emit = (line: string) => {
    if (!line) return false;
    const message = JSON.parse(line) as StreamMessage;
    onMessage(message);
    return message.type === "done";
  };

  try {
    for (;;) {
      const { value, done } = await reader.read();

      if (done) {
        buffer += decoder.decode();
        if (buffer.endsWith("\r")) buffer = buffer.slice(0, -1);
        if (buffer) emit(buffer);
        return;
      }

      buffer += decoder.decode(value, { stream: true });
      const lines = buffer.split("\n");
      buffer = lines.pop() ?? "";

      for (const rawLine of lines) {
        const line = rawLine.endsWith("\r") ? rawLine.slice(0, -1) : rawLine;
        if (emit(line)) {
          await reader.cancel();
          return;
        }
      }
    }
  } finally {
    reader.releaseLock();
  }
}
```

`buffer` 保留未闭合的半行，`TextDecoder` 保留未完成的 UTF-8 字节。解析 SSE 时，读取与解码部分不变，分帧器改为按空行聚合 field lines。

## 链路缓冲

流式响应要求整条链路持续放行：

1. Server 每次写入后及时 flush。
2. 框架与压缩中间件不聚合小块输出。
3. 反向代理关闭响应缓冲。
4. Client 增量读取 `response.body`，避免调用 `json()` 或 `text()`。

nginx 的 `proxy_buffering` 默认开启。流式路由可以显式配置：

```nginx
location /stream {
  proxy_pass http://app;
  proxy_buffering off;
  proxy_read_timeout 5m;
}
```

后端也可以返回：

```http
X-Accel-Buffering: no
```

心跳间隔需要短于代理的 idle timeout。修改 MIME 无法替代 flush 与 buffering 配置。

## 背压与取消

`ReadableStream` 通过内部队列与 `desiredSize` 传播背压信号。这个机制可以约束浏览器中的 pipe chain；信号能否继续影响远端 Server，取决于底层网络与服务端实现。

页面离开或收到终止事件时，应取消读取：

```ts
const controller = new AbortController();

fetch("/stream", { signal: controller.signal });
controller.abort();
```

仅停止 UI 更新，连接仍会占用网络与服务端资源。
