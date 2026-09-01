---
title: Server-Sent Events（SSE）
date: 2024-11-01 12:16:30
categories:
  - 技术
tags:
  - 网络
---

SSE（Server-Sent Events）是 Server → Client 的 HTTP 事件流格式，MIME 为 `text/event-stream`。Server 保持响应打开并持续写入事件；浏览器可用 `EventSource` 完成解析、派发与重连。Client → Server 仍使用普通 HTTP 请求。

<!-- more -->

## 数据格式

用 `\n` 表示换行。事件由若干字段行组成，以空行结束：

```text
event: userupdate\ndata: {"uid":1}\nid: 98769879675\nretry: 10000\n\n
```

规范允许 `\r\n`、`\n`、`\r` 三种行尾。服务端通常统一输出 `\n`；客户端解析器应兼容三种形式。

每一行按第一个 `:` 分成 field 与 value：

- value 开头的一个空格会被移除；更多空格保留
- `:` 开头的行是注释，解析器直接忽略
- 没有 `:` 的行，整行作为 field，value 为空串
- 未知 field 直接忽略
- 整个流使用 UTF-8；开头允许一个 BOM

标准字段：

- `data`：事件数据；多行 `data` 用 `\n` 合并
- `event`：事件类型；缺省时为 `message`
- `id`：更新 last event ID，供断线重连使用
- `retry`：建议浏览器下一次重连前等待的毫秒数

`data:` 这一行可以没有内容。只要出现过 `data` field，空行到来时仍会派发事件；完全没有 `data` field 的块不会派发事件。

```text
data: first line\n
data: second line\n
\n
```

对应的 `data` 为 `first line\nsecond line`。

流关闭时，末尾没有空行的半个事件会被丢弃。Server 必须为最后一个事件补上空行。

## 服务端生成事件

```ts
type SSEMessage = {
  event?: string;
  data: string;
  id?: string;
  retry?: number;
};

function encodeSSE(message: SSEMessage): Uint8Array {
  const lines: string[] = [];

  if (message.event !== undefined) lines.push(`event: ${message.event}`);
  if (message.id !== undefined) lines.push(`id: ${message.id}`);
  if (message.retry !== undefined) lines.push(`retry: ${message.retry}`);

  // data 内的换行必须拆成多行；客户端会用 \n 合并。
  for (const line of message.data.split(/\r\n|\r|\n/)) {
    lines.push(`data: ${line}`);
  }

  lines.push("", "");
  return new TextEncoder().encode(lines.join("\n"));
}
```

写入响应后需要及时 flush。应用框架、压缩中间件、反向代理或 CDN 任意一段缓冲，都可能让多个事件积累后一起到达。

## 消息类型

省略 `event` 时，浏览器触发 `message`：

```text
data: {"status":"ready"}\n\n
```

指定 `event` 后，可按事件类型注册处理器：

```text
event: userupdate\n
data: {"uid":1}\n
\n
```

```js
const source = new EventSource("/events");

source.onmessage = (event) => {
  console.log(event.data);
};

source.addEventListener("userupdate", (event) => {
  console.log(JSON.parse(event.data));
});
```

## 心跳

HTTP 连接可能被反向代理、负载均衡器或 NAT 按空闲时间关闭。SSE 使用注释行发送应用层心跳：

```text
:\n\n
: keep-alive\n\n
```

注释不会触发事件，只会让链路持续出现字节。心跳间隔应短于整条链路中最短的 idle timeout；它与 TCP keepalive 解决的层级不同。

SSE 是单向通道，心跳没有回执。需要确认 Client 在线时，应另行设计双向通信或确认请求。

## 断线重连

`EventSource` 内置重连流程：

1. 连接中断后进入 `CONNECTING`。
2. 等待重连时间；Server 可用 `retry: <ms>` 更新这个值。
3. last event ID 非空时，新请求自动携带 `Last-Event-ID`。
4. Server 根据该 ID 从断点继续发送。

```text
id: 42\n
data: {"status":"processing"}\n
retry: 3000\n
\n
```

Server 必须保存事件 ID 与可重放数据，`Last-Event-ID` 才能形成真正的断点续传。只发送 `id:`、没有 `data:` 的块不会触发事件，但会推进 last event ID。

## EventSource API

```js
const source = new EventSource("/events", { withCredentials: true });

source.onopen = () => console.log("connected");
source.onerror = () => console.log(source.readyState);
source.onmessage = (event) => {
  console.log(event.lastEventId, event.data);
};

source.close();
```

`EventSource` 负责解析 SSE、派发事件和自动重连。限制：

- 请求方法固定为 GET
- 构造参数只能控制 URL 与 `withCredentials`
- 无法添加 `Authorization` 等自定义请求头
- 响应必须是 HTTP 200 + `text/event-stream`

需要 POST、请求体或自定义 Header 时，可用 `fetch()` 发起请求，从 `response.body` 读取字节并解析 SSE。事件格式保持不变；重连与 `Last-Event-ID` 由应用实现。
