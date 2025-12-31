---
title: 流式响应（Chat）：HTTP 响应体字节流的读取与解析
date: 2025-12-29 20:02:01
categories:
  - 技术
tags:
  - 网络
---

> 主要通过 web fetch api 的 `ReadableStream` 能力，解释 HTTP 通道中的流式响应。
> 流式关键在于 HTTP 协议层解析 Body 体，和 TCP 粘包和拆包的处理类似。各种编程语言都具有 body 二进制数据流的拦截和解析能力。

## Fetch API 速览

- `await fetch(url)` → 返回 `Response` 对象：这时浏览器通常已经从底层连接（TCP/QUIC）里拿到并解析完 **HTTP 响应行（status）+ 响应头（headers）**；但 **响应体（body）** 还没被消费/解析（它会通过单独的流式接口暴露出来，供上层代码增量读取）。
- `await res.json()` / `await res.text()` → **一次性读取并解析完整 Body**，适用于非流式场景。
- `res.body` → `ReadableStream<Uint8Array>`，**字节流接口**，可增量消费。
- `res.body.getReader().read()` → 手动 Pull 模式，每次返回 `{ value: Uint8Array, done: boolean }`。

可以这样理解：`fetch()` 先把 **status/headers** 拿到手并封装成 `Response`；至于 **body 字节** 怎么读、怎么解析，由调用方选择：

- 通过 `ReadableStream` 增量读取（适合 AI token、NDJSON、SSE 等流式场景）。
- 或调用 `json()` / `text()` 这类封装好的方法，让它们内部把 body 全部读完后再一次性解析。

**核心区别**：`res.json()` 等“便捷方法”会等待整个响应体下载完毕后一次性解析；`reader.read()` 则支持**边接收边处理**，是流式读取的基础。

<!-- more -->

```ts
// 非流式：等 Body 全部接收完再解析
const full = await (await fetch(url)).json();

// 流式：边接收边处理（Chunk 边界不等于消息边界）
const res = await fetch(url);
const reader = res.body!.getReader();
const decoder = new TextDecoder("utf-8");
for (;;) {
  const { value, done } = await reader.read();
  if (done) break;
  const chunkText = decoder.decode(value, { stream: true });
  // handle(chunkText)
}
```

## SSE 与 Fetch

很多“Chat 流式输出”看起来像 Server-Sent Events (SSE)，但底层实现常见就两类：

- **SSE（协议 + API）**：`text/event-stream` + `EventSource`（浏览器负责按协议分帧并提供重连等语义）。参考 [SSE 指南](https://www.yigegongjiang.com/2024/sse/)。
- **Fetch 响应体流式读取（传输能力 + 自定义分帧）**：`fetch()` + `Response.body`（拿到的是 `ReadableStream<Uint8Array>`，需要自己定义消息边界与语义，如 NDJSON/长度前缀等）。
- 顺带一提：很多实现也在从 SSE/`EventSource` 转向 `fetch` stream，因为 SSE 本身有不少限制（比如只能 GET、Header/鉴权不够灵活等）。

共同点：两者都依赖“长连接 + 增量写入 + 及时 Flush”，最后在客户端看起来都是 **HTTP 响应体字节流**。

一句话区别：**SSE 把“怎么切消息 + 事件语义”都标准化了；Fetch 流式读取只把字节暴露给应用层，剩下由应用协议来定。**

- **分帧**：SSE 固定是 `data:` 行 + 空行；Fetch Stream 的边界完全自定义（NDJSON、长度前缀、分隔符等）。
- **语义**：SSE 浏览器内建重连、`Last-Event-ID`；Fetch Stream 想要重连/断点续传/错误语义，需要在应用层设计。
- **适用场景**：SSE 更像“标准事件流”；而 Chat 经常需要 POST、鉴权 Header、以及自定义协议时，Fetch Stream 会更顺手。

## 链路传输（从服务端 write 到客户端拿到 chunk）

想要真的“边推边显示”，关键往往不是 HTTP 语法本身，而是：**字节在链路的哪一段被缓冲住了**。

1. **服务端应用写出**：应用调用 `write()`/`send()` 将字节写入 Socket。若仅写入用户态缓冲而未执行 `flush`（或被框架/中间件缓冲），客户端将无法接收增量数据。
2. **TCP Socket 缓冲区（发送/接收）**：`send()` 通常只是把数据拷贝进 **TCP 发送缓冲区**，真正“发出去”要看 TCP 栈怎么分段、流控/拥塞控制允不允许。结果就是：**应用 write 的粒度**，基本不等于 **对端 read 到的粒度**（还会受 Nagle/延迟 ACK/cwnd/rwnd 等影响）。
3. **HTTP 承载方式**：
   - **HTTP/1.1**：通常使用分块传输编码（Chunked Transfer Encoding）或在未知 `Content-Length` 时持续写入。
   - **HTTP/2 / HTTP/3**：基于 DATA Frame 或 QUIC Stream 持续传输，受多路复用与流控机制影响。
4. **浏览器网络栈 → ReadableStream**：浏览器将“已到达且可用”的字节推入 `ReadableStream` 内部队列，JavaScript 通过 `reader.read()` 或 `pipeThrough()` 以 Pull 模式消费。

- **Chunk 边界 ≠ 消息边界**：一次 `read()` 获取的 `Uint8Array` 仅是当前可用的字节片段，可能截断在任意位置（如 UTF-8 字符中间、JSON 结构中间或自定义帧头中间）。
- **全链路缓冲会“假装不流式”**：应用层 Flush、反向代理 Buffering、压缩器缓冲、CDN 策略、浏览器内部队列……任何一段在攒数据，都会让 Token 看起来变成“凑一批才到”。

## Fetch 流式读取的基本模型

`fetch()` 返回的 `Response` 对象包含 `body` 属性，其类型为 `ReadableStream<Uint8Array>`。消费方式主要有两种：

1. **手动读取**：获取 `reader` 并循环调用 `read()`。
2. **管道处理**：使用 `pipeThrough()` 构建解码、分帧、解析的流水线（推荐）。

整体可以当成 **pull 模式**：每次 `read()` 拿到的是“目前已经到手的那点字节”。如果处理速度慢于网络进入速度，队列就会堆起来，进而触发 **背压（Backpressure）**（后续传输会被放慢）。

## 示例：解析消息流

### 字节解码：处理增量 UTF-8

网络传输交付的是 `Uint8Array`，而 Chat 最终要的是文本 Token。注意 UTF-8 是变长编码，一个字符可能被拆到两个 Chunk 里；如果直接 `decoder.decode(chunk)`（默认非流式），边界处就可能乱码/丢字。这里要用增量解码：

- 使用 `TextDecoder` 的 `{ stream: true }` 选项。
- 或使用 `TextDecoderStream` 管道：`response.body.pipeThrough(new TextDecoderStream())`，直接获得 `ReadableStream<string>`。

### 分帧策略：定义消息边界

想做到“边接收边渲染”，需要先定一个能增量解析的分帧（Framing）规则：到底每条消息怎么切出来？

1. **NDJSON / JSON Lines（推荐）**
   - 格式：每条消息占一行，如 `{"type":"delta","text":"..."}\n`。
   - 优点：解析简单，调试友好，兼容 `JSON.parse`。
   - 注意：需确保 Payload 内无未转义的换行符（标准 JSON 字符串会将换行编码为 `\n`，通常安全）。

2. **分隔符协议**
   - 格式：使用自定义分隔符（如 `\n\n` 或特定 Boundary）切分消息。
   - 风险：若 Payload 包含分隔符，需进行转义或设计复杂的 Boundary 机制。

3. **长度前缀（二进制 Framing）**
   - 格式：`[Length][Payload]...`。
   - 优点：对任意二进制或文本内容安全，不受内容字符影响。
   - 缺点：实现复杂度较高，需维护字节级状态机。

一般优先选 **NDJSON** 或 **长度前缀**；千万别指望 Chunk 边界刚好对齐业务消息边界。

### Fetch + NDJSON

**字节读取 → 文本解码 → 行分帧 → JSON 解析 → UI 更新** 的完整流程：

```typescript
type ChatChunk = { type: "delta"; text: string } | { type: "done" } | { type: "error"; message: string };

export async function streamChat(
  input: { prompt: string },
  onChunk: (c: ChatChunk) => void,
  signal?: AbortSignal,
) {
  const res = await fetch("/api/chat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(input),
    signal,
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  if (!res.body) throw new Error("ReadableStream not supported");

  // 1. 字节读取层：获取 ReadableStream reader
  const reader = res.body.getReader();

  // 2. 文本解码层：处理 UTF-8 边界
  const decoder = new TextDecoder("utf-8");

  // 3. 行分帧层：缓冲未闭合的行
  let lineBuffer = "";

  try {
    while (true) {
      // 读取下一批字节（Chunk 边界随机）
      const { value, done } = await reader.read();

      if (done) {
        // 流结束，处理缓冲区剩余内容
        if (lineBuffer.trim()) {
          const msg = JSON.parse(lineBuffer.trim()) as ChatChunk;
          onChunk(msg);
        }
        break;
      }

      // 增量解码：stream: true 保留不完整字节序列
      const text = decoder.decode(value, { stream: true });
      lineBuffer += text;

      // 按换行符切分
      const lines = lineBuffer.split("\n");
      // 最后一个元素可能是不完整行，留待下轮处理
      lineBuffer = lines.pop() ?? "";

      // 4. JSON 解析层：逐行解析
      for (const line of lines) {
        const trimmed = line.trim();
        if (!trimmed) continue; // 跳过空行

        const msg = JSON.parse(trimmed) as ChatChunk;
        onChunk(msg);

        // 5. 业务终止：显式结束信号
        if (msg.type === "done") return;
      }
    }
  } catch (e) {
    throw e;
  } finally {
    reader.releaseLock();
  }
}
```

**服务端 send**：

> **将增量 Token 封装为可切分的消息单元（行），确保客户端始终解析完整的 JSON 对象。**

- 增量 Token：`{"type":"delta","text":"..."}\n` + Flush
- 结束信号：`{"type":"done"}\n` + End Response

---
