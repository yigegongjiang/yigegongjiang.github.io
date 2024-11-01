---
title: SSE 指南
date: 2024-11-01 12:16:30
categories:
- 技术
tags:
- 网络
---

Server-Sent Events（SSE）是一种允许服务器通过HTTP连接主动向客户端发送数据的技术。它主要被用于创建实时应用，如消息推送和实时通知。SSE 使用简单的文本格式发送消息，这种格式使得其易于在浏览器中实现和使用。

SSE 注意事项：
1. 通过 HTTP 协议通道 建立单向长连接，即 Client 连接 Server 后，Server 不断开连接，并持续的通过 socket 套接字发送 data 给 Client。
2. 网关等设备会主动关闭 tcp 通道，需要 SSE Server 端增加心跳。很多种场景都会导致 tcp 连接中断，和 IM 心跳一致。这里需要 SSE Server 增加应用层心跳，非 TCP 层心跳。

<!-- more -->

# 数据格式

通过具有明显分割线的消息体，来分割数据字段：

```
// 以下文本消息体，最终通过编译成二进制的形式被传递和解析，通过 \n 等标记符号进行【行】分割。

event: message/userupdate/custom...\n
data: {xxx}\n
id: 98769879675\n
retry: 10000\n
\n

// 一个完整的消息体如下，通过 \n 分割行，末尾通过 \n\n 分割单个消息体
id: event-id-1\ndata:event-data-first\n\n
```

以上 event/data/id/retry 四个字段中，data 是必须字段，其他三个是可选字段。每个消息体，必须以 \n\n 作为末尾标记。

在 ts 中，可行的生成消息体的 code 如下：

```
  private encodeMessage(message: ChatMessage) {
    const data = {
      type: message.type,
      payload: message.data,
    };
    const content = [
      `id: ${message.id}`,
      `data: ${JSON.stringify(data)}`,
      '',
      '',
    ].join('\n'); // 这里，通过空行分割，在消息体的末尾增加 `\n\n` 标记
    return this.encoder.encode(content);
  }
```

对消息体的解析，也同样遵循固定的规律，即对二进制中的 \n 和 \n\n 进行解析。

解析流程：

```
1. 通过 \n\n 对消息体进行分割，获得一个消息体的二进制内容并开始解析
2. 通过 \n 对行进行解析，获得 xxx:xxx 这样的一行内容
3. 通过 : 符号，对行进行解析，获得 key:xxx 和 value:xxx 两个内容
4. 整合获得的所有内容，聚合成 {id:xxx,event:xxx,data:xxx,retry:xxx} 这样的消息体
```

# 消息类型

SSE 通过 event 字段，可以自定义各种消息体。约定通用的消息体是 message。有如下两种消息体：

```
id: xxx\ndata:xxx\nevent:message\n\n
id:xxx\ndata:xxx\nevent:{custom-name}\n\n
```

如果 event 为空，默认当作且应该当作 message 消息来解析和处理。

# 重连

http 通道可能发生中断，每条消息题都可以携带一个 retry 字段，用来告知 client 在断开后多久应该重连。

而重连的逻辑就是重新发起 http 连接。这里为了保持和之前的通道一致，应该在重连的时候在 header 中携带最后一次收到的消息的 id，可以让 server 侧知道从哪里中断的，以保持连续的服务。

# 心跳

SSE 应该使用应用层心跳，即发送一条空消息体，以保持 http tcp 套接字不被网关、nat 等场景强制关闭。格式如下：

```
:\n\n
:heartbeat\n\n
```

这里，心跳可以不遵循消息体的约定。

之前提到消息体一定需要有 data 字段，因为每一条消息都是需要传递信息，如果没有 data 字段，那么这条消息就无法被解析，也就没有传递的必要。

但是这套约定，并不是说没有data字段，消息的发送、接收、解析 整套流程就会失败，因为对于 TCP/UDP/HTTP 这套协议来说，它并不关心消息体的内容是什么。

而 Client 端，对消息体的解析应该是包容的，即消息体如果不符合约定，那么就应该抛弃。

这里的应用层心跳，就会走到这套逻辑里面。IM 是双向通信，为了保障心跳的到达，Client 端需要解析完整的心跳并回执。在 SSE 单向通道里面，只需要保障有一条消息从 Server 发往 Client 即可（没有成功率保障，因为是单向通信）。

___


