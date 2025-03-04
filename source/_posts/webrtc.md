---
title: WebRTC 的一些解释
date: 2024-11-09 15:45:07
categories:
- 技术
tags:
- 网络
- 音视频
---

最近工作上在使用 WebRTC，在公司内部做了技术分享。这里把内容进行脱敏，整理公开。

# 快速进入 WebRTC

![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/20241109160331..png)

<!-- more -->

## 媒体协商 概要

![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091611671.png)

1. 每方确定自己的编解码、网络等信息（媒体）
2. 每方将自己的信息给到对方（协商）

## 信令 概要
webrtc 建联前，在各方之间传输【协商数据】
* offer、answer、candidate

webrtc 建联后，在各方之间传输【控制信息】
* mute、leave、join、…

# 多样化应用场景
webrtc 通道多样：
* 每条 webrtc 通道连接两个端
* 每个端可以有 n 个webrtc 通道
* 服务器可以成为其中一个端

对【端】的影响：
* 上行 和 下行带宽
* 视频编解码对 CPU/GPU 的消耗

2 人参会，两个【客户端】直接打通，最舒适
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091616249.png)
4 人参会，每个【客户端】打开多个通道，上下行剧增，硬件消耗剧增
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091617239.png)
8 人参会，【服务端】作为中转通道，相比 4 人场景，下行剧增，上行剧减，硬件消耗增加
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091617755.png)
50 人参会，【服务端】作为终极合流通道。相比 8 人场景，客户端回退到 2 人消耗，并把所有消耗转移到【服务端】
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091618093.png)
Tips：当【服务端】作为 webrtc 的一个通道后，玩法就非常多样了。比如【说话人大屏】、【服务端滤镜】等

# 流媒体的质量约束
## 清晰度 vs 流畅度 vs 码率
分辨率影响清晰度，越大越好，流尺寸也会越大。
> 补充：同样分辨率在不同的显示屏上，也会有清晰度的差异。详见 [分辨率解释](https://www.yigegongjiang.com/2024/px/)，介绍了【光源】、【虚拟像素】、【像素软件化】、【高清屏】对图片的影响。

帧率影响流畅度。视频由 帧 画面组成，同一秒内 帧 画面越多，肉眼感知越流畅，流尺寸也会越大。
> 电影：24fps，电视机：30 fps，电脑 & 手机：60 fps - 120 fps

码率：每秒传输的比特数据量。单位：[number] bps
> 码率 = 分辨率 × 比特深度 × 帧率 × 压缩比
> 比特深度：描述一个 px 所需要的 bit。通常 RGB 为 3 色彩通道，每个色彩需要 8 个 bit，即 24 bit。
> 压缩比：H.264/H.265 压缩效率极高，根据视频画面的【前后帧之间的 px 差异】，极大降低视频流尺寸。
> 
> 示例：（场景设定：分辨率：1080p，色彩通道：3，帧率：30，编码：H.264）
> 码率 = 1920 × 1080 × 24 × 30 x  1/100 = 14.93 Mbps = 14930Kbps = 1866.25KBps = 1.87 MBps
> 即：每秒需要 1.87 M 的流量

在 webrtc 中，分辨率和帧率都是由开发人员控制的（主动调用 sdk api 将 stream 给到 sdk）。其中：
1. stream 本身具有尺寸，即 分辨率。
2. 调用 sdk api 的频次，即 帧率。

webrtc 控制码率有两种行为：
1. 主动控制：当发现网络状态不佳，webrtc 会通过内部算法自行调整码率，包括自行调整码率，甚至直接丢弃数据包。
2. 被动约束：开发人员预设期望码率范围，但【主动控制】优先。

## 编解码
H.264/5 等编码。很重要，但还不会。这里留坑。

# 媒体协商的多样性

## SDP 是什么
SDP - Session Description Protocol（会话描述协议）。用于在两个通信终端之间描述细节。
> 强制依赖【协议描述规则】进行解析

a - b 之间约定了 n 条通信规则，就有 n 种 SDP 通信数据。

## SDP - offer/answer
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091622792.png)

### SDP Format:
```
示例//=============会话描述====================
v=0
o=- 7017624586836067756 2 IN IP4 127.0.0.1
s=-
t=0 0
...

//================媒体描述=================
//================音频媒体=================
/*
 * 音频使用端口1024收发数据
 * UDP/TLS/RTP/SAVPF 表示使用 dtls/srtp 协议对数据加密传输
 * 111、103 ... 表示本会话音频数据的 Payload Type
 */
 m=audio 1024 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 126

//==============网络描述==================
//指明接收或者发送音频使用的IP地址，由于WebRTC使用ICE传输，这个被忽略。
c=IN IP4 0.0.0.0
//用来设置rtcp地址和端口，WebRTC不使用
a=rtcp:9 IN IP4 0.0.0.0
...

//==============音频安全描述================
//ICE协商过程中的安全验证信息
a=ice-ufrag:khLS
a=ice-pwd:cxLzteJaJBou3DspNaPsJhlQ
a=fingerprint:sha-256 FA:14:42:3B:C7:97:1B:E8:AE:0C2:71:03:05:05:16:8F:B9:C7:98:E9:60:43:4B:5B:2C:28:EE:5C:8F3:17
...

//==============音频流媒体描述================
a=rtpmap:111 opus/48000/2
//minptime代表最小打包时长是10ms，useinbandfec=1代表使用opus编码内置fec特性
a=fmtp:111 minptime=10;useinbandfec=1
...
a=rtpmap:103 ISAC/16000
a=rtpmap:104 ISAC/32000
a=rtpmap:9 G722/8000
...

//=================视频媒体=================
m=video 9 UDP/TLS/RTP/SAVPF 100 101 107 116 117 96 97 99 98
...
//=================网络描述=================
c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0
...
//=================视频安全描述=================
a=ice-ufrag:khLS
a=ice-pwd:cxLzteJaJBou3DspNaPsJhlQ
a=fingerprint:sha-256 FA:14:42:3B:C7:97:1B:E8:AE:0C2:71:03:05:05:16:8F:B9:C7:98:E9:60:43:4B:5B:2C:28:EE:5C:8F3:17
...

//================视频流描述===============
a=mid:video
...
a=rtpmap:100 VP8/90000
//================服务质量描述===============
a=rtcp-fb:100 ccm fir
a=rtcp-fb:100 nack //支持丢包重传，参考rfc4585
a=rtcp-fb:100 nack pli
a=rtcp-fb:100 goog-remb //支持使用rtcp包来控制发送方的码流
a=rtcp-fb:100 transport-cc
...
```

### SDP Raw Example:
```
v=0
o=- 5595748951276187774 2 IN IP4 127.0.0.1
s=-
t=0 0
a=group:BUNDLE 0 1 2
a=extmap-allow-mixed
a=msid-semantic: WMS FAMS
m=audio 9 UDP/TLS/RTP/SAVPF 111 63 9 102 0 8 13 110 126
c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0
a=ice-ufrag:2D6J
a=ice-pwd:jWsbseLplx44YJwXRravag7Y
a=ice-options:trickle renomination
a=fingerprint:sha-256 F4:57:8B:7E:6C:A1:0F:08:95:72:67:68:3C:18:67:40:39:D9:D5:0A:A4:6E:C0:3E:67:BA:E0:02:B5:FF:EF:40
a=setup:actpass
a=mid:0
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
a=extmap:2 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
a=extmap:3 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01
a=extmap:4 urn:ietf:params:rtp-hdrext:sdes:mid
a=sendrecv
a=msid:FAMS FAMSa0
a=rtcp-mux
a=rtpmap:111 opus/48000/2
a=rtcp-fb:111 transport-cc
a=fmtp:111 minptime=10;useinbandfec=1
a=rtpmap:63 red/48000/2
a=fmtp:63 111/111
a=rtpmap:9 G722/8000
a=rtpmap:102 ILBC/8000
a=rtpmap:0 PCMU/8000
a=rtpmap:8 PCMA/8000
a=rtpmap:13 CN/8000
a=rtpmap:110 telephone-event/48000
a=rtpmap:126 telephone-event/8000
a=ssrc:3316953618 cname:cXfVAjx41LASIlBT
a=ssrc:3316953618 msid:FAMS FAMSa0
m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 127
c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0
a=ice-ufrag:2D6J
a=ice-pwd:jWsbseLplx44YJwXRravag7Y
a=ice-options:trickle renomination
a=fingerprint:sha-256 F4:57:8B:7E:6C:A1:0F:08:95:72:67:68:3C:18:67:40:39:D9:D5:0A:A4:6E:C0:3E:67:BA:E0:02:B5:FF:EF:40
a=setup:actpass
a=mid:1
a=extmap:14 urn:ietf:params:rtp-hdrext:toffset
a=extmap:2 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
a=extmap:13 urn:3gpp:video-orientation
a=extmap:3 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01
a=extmap:5 http://www.webrtc.org/experiments/rtp-hdrext/playout-delay
a=extmap:6 http://www.webrtc.org/experiments/rtp-hdrext/video-content-type
a=extmap:7 http://www.webrtc.org/experiments/rtp-hdrext/video-timing
a=extmap:8 http://www.webrtc.org/experiments/rtp-hdrext/color-space
a=extmap:4 urn:ietf:params:rtp-hdrext:sdes:mid
a=extmap:10 urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id
a=extmap:11 urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id
a=sendonly
a=msid:FAMS FAMSv1
a=rtcp-mux
a=rtcp-rsize
a=rtpmap:96 H264/90000
a=rtcp-fb:96 goog-remb
a=rtcp-fb:96 transport-cc
a=rtcp-fb:96 ccm fir
a=rtcp-fb:96 nack
a=rtcp-fb:96 nack pli
a=fmtp:96 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=640c34
a=rtpmap:97 rtx/90000
a=fmtp:97 apt=96
a=rtpmap:98 H264/90000
a=rtcp-fb:98 goog-remb
a=rtcp-fb:98 transport-cc
a=rtcp-fb:98 ccm fir
a=rtcp-fb:98 nack
a=rtcp-fb:98 nack pli
a=fmtp:98 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42e034
a=rtpmap:99 rtx/90000
a=fmtp:99 apt=98
a=rtpmap:100 red/90000
a=rtpmap:101 rtx/90000
a=fmtp:101 apt=100
a=rtpmap:127 ulpfec/90000
a=ssrc-group:FID 439044504 650966991
a=ssrc:439044504 cname:cXfVAjx41LASIlBT
a=ssrc:439044504 msid:FAMS FAMSv1
a=ssrc:650966991 cname:cXfVAjx41LASIlBT
a=ssrc:650966991 msid:FAMS FAMSv1
m=application 9 UDP/DTLS/SCTP webrtc-datachannel
c=IN IP4 0.0.0.0
a=ice-ufrag:2D6J
a=ice-pwd:jWsbseLplx44YJwXRravag7Y
a=ice-options:trickle renomination
a=fingerprint:sha-256 F4:57:8B:7E:6C:A1:0F:08:95:72:67:68:3C:18:67:40:39:D9:D5:0A:A4:6E:C0:3E:67:BA:E0:02:B5:FF:EF:40
a=setup:actpass
a=mid:2
a=sctp-port:5000
a=max-message-size:262144
```

### SDP Raw Example Desc:
```
//==会话描述==

v=0
- **协议版本**：SDP 协议版本号，这里是版本0。

o=- 5595748951276187774 2 IN IP4 127.0.0.1
- **会话起源**：
  - 用户名：`-`（匿名）
  - 会话ID：`5595748951276187774`
  - 会话版本：`2`
  - 网络类型：`IN`（互联网）
  - 地址类型：`IP4`
  - 地址：`127.0.0.1`（本地回环地址）

s=-
- **会话名称**：`-`（未指定）

t=0 0
- **会话活动时间**：从`0`到`0`，表示会话无限期有效。

a=group:BUNDLE 0 1 2
- **媒体流分组**：使用`BUNDLE`机制将媒体流`0`、`1`和`2`绑定在一起，共享一个传输通道。

a=extmap-allow-mixed
- **扩展映射混合允许**：允许同时使用一字节和两字节的RTP头扩展（参考RFC 8285）。

a=msid-semantic: WMS FAMS
- **媒体流标识语义**：`WMS`表示WebRTC媒体流，`FAMS`是媒体流的标识符。


//==音频媒体描述==

m=audio 9 UDP/TLS/RTP/SAVPF 111 63 9 102 0 8 13 110 126
- **媒体描述**：
  - 媒体类型：`audio`（音频）
  - 端口：`9`（通常为`9`表示使用UDP/TLS/RTP协议并通过ICE协商实际端口）
  - 传输协议：`UDP/TLS/RTP/SAVPF`（安全的RTP协议，带反馈机制）
  - 有效负载类型（编码）：`111`、`63`、`9`、`102`、`0`、`8`、`13`、`110`、`126`

c=IN IP4 0.0.0.0
- **连接信息**：
  - 网络类型：`IN`（互联网）
  - 地址类型：`IP4`
  - 地址：`0.0.0.0`（占位符，表示实际地址通过ICE协商）

a=rtcp:9 IN IP4 0.0.0.0
- **RTCP连接信息**：
  - 端口：`9`
  - 网络类型：`IN`
  - 地址类型：`IP4`
  - 地址：`0.0.0.0`

a=ice-ufrag:2D6J
- **ICE用户名碎片**：`2D6J`，用于ICE候选者的身份认证。

a=ice-pwd:jWsbseLplx44YJwXRravag7Y
- **ICE密码**：`jWsbseLplx44YJwXRravag7Y`，用于ICE身份验证。

a=ice-options:trickle renomination
- **ICE选项**：
  - `trickle`：支持逐步收集和发送ICE候选者（Trickle ICE）。
  - `renomination`：支持重新提名最佳候选者。

a=fingerprint:sha-256 F4:57:8B:7E:6C:A1:0F:08:95:72:67:68:3C:18:67:40:39:D9:D5:0A:A4:6E:C0:3E:67:BA:E0:02:B5:FF:EF:40
- **DTLS指纹**：
  - 哈希算法：`sha-256`
  - 指纹值：用于验证DTLS握手的安全性。

a=setup:actpass
- **DTLS连接角色**：`actpass`表示主动-被动，双方可协商谁主动发起DTLS握手。

a=mid:0
- **媒体标识符**：`0`，用于在BUNDLE中标识此媒体流。

a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
- **RTP头扩展**：
  - ID：`1`
  - URI：`ssrc-audio-level`，用于传输音频电平信息。

a=extmap:2 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
- **RTP头扩展**：
  - ID：`2`
  - URI：`abs-send-time`，用于同步媒体流的发送时间。

a=extmap:3 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01
- **RTP头扩展**：
  - ID：`3`
  - URI：`transport-wide-cc`，用于传输层的拥塞控制。

a=extmap:4 urn:ietf:params:rtp-hdrext:sdes:mid
- **RTP头扩展**：
  - ID：`4`
  - URI：`mid`，用于标识媒体流的MID。

a=sendrecv
- **发送接收方向**：支持同时发送和接收媒体流。

a=msid:FAMS FAMSa0
- **媒体流标识**：
  - 流ID：`FAMS`
  - 轨道ID：`FAMSa0`

a=rtcp-mux
- **RTCP复用**：音频的RTCP流与RTP流复用在同一个端口。

a=rtpmap:111 opus/48000/2
- **RTP映射**：
  - 编码：`opus`
  - 采样率：`48000`Hz
  - 声道数：`2`

a=rtcp-fb:111 transport-cc
- **RTCP反馈机制**：
  - 针对编码`111`（opus），支持`transport-cc`（传输层拥塞控制）。

a=fmtp:111 minptime=10;useinbandfec=1
- **格式参数**：
  - 针对编码`111`（opus）
  - `minptime=10`：最小包时间为10毫秒
  - `useinbandfec=1`：启用带内前向纠错（FEC），提高音频质量

a=rtpmap:63 red/48000/2
- **RTP映射**：
  - 编码：`red`（冗余编码）
  - 采样率：`48000`Hz
  - 声道数：`2`

a=fmtp:63 111/111
- **格式参数**：
  - 针对编码`63`（red）
  - `111/111`：表示冗余编码中使用的主、次编码类型都是`111`（opus）

a=rtpmap:9 G722/8000
- **RTP映射**：
  - 编码：`G722`
  - 采样率：`8000`Hz

a=rtpmap:102 ILBC/8000
- **RTP映射**：
  - 编码：`iLBC`
  - 采样率：`8000`Hz

a=rtpmap:0 PCMU/8000
- **RTP映射**：
  - 编码：`PCMU`（G.711 μ-law）
  - 采样率：`8000`Hz

a=rtpmap:8 PCMA/8000
- **RTP映射**：
  - 编码：`PCMA`（G.711 A-law）
  - 采样率：`8000`Hz

a=rtpmap:13 CN/8000
- **RTP映射**：
  - 编码：`CN`（舒适噪声）
  - 采样率：`8000`Hz

a=rtpmap:110 telephone-event/48000
- **RTP映射**：
  - 编码：`telephone-event`（DTMF信号）
  - 采样率：`48000`Hz

a=rtpmap:126 telephone-event/8000
- **RTP映射**：
  - 编码：`telephone-event`（DTMF信号）
  - 采样率：`8000`Hz

a=ssrc:3316953618 cname:cXfVAjx41LASIlBT
- **同步源（SSRC）标识**：
  - SSRC：`3316953618`
  - CNAME：`cXfVAjx41LASIlBT`，用于跨媒体流同步

a=ssrc:3316953618 msid:FAMS FAMSa0
- **媒体流标识**：
  - 流ID：`FAMS`
  - 轨道ID：`FAMSa0`

//==视频媒体描述==

m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 127
- **媒体描述**：
  - 媒体类型：`video`（视频）
  - 端口：`9`（通过ICE协商实际端口）
  - 传输协议：`UDP/TLS/RTP/SAVPF`（安全的RTP协议，带反馈机制）
  - 有效负载类型（编码）：`96`、`97`、`98`、`99`、`100`、`101`、`127`

c=IN IP4 0.0.0.0
- **连接信息**：与音频部分相同

a=rtcp:9 IN IP4 0.0.0.0
- **RTCP连接信息**：与音频部分相同

a=ice-ufrag:2D6J
- **ICE用户名碎片**：`2D6J`，与音频部分相同

a=ice-pwd:jWsbseLplx44YJwXRravag7Y
- **ICE密码**：`jWsbseLplx44YJwXRravag7Y`，与音频部分相同

a=ice-options:trickle renomination
- **ICE选项**：支持Trickle ICE和重新提名机制

a=fingerprint:sha-256 F4:57:8B:7E:6C:A1:0F:08:95:72:67:68:3C:18:67:40:39:D9:D5:0A:A4:6E:C0:3E:67:BA:E0:02:B5:FF:EF:40
- **DTLS指纹**：用于DTLS握手的安全验证

a=setup:actpass
- **DTLS连接角色**：双方均可主动或被动，待协商

a=mid:1
- **媒体标识符**：`1`，用于BUNDLE中的媒体流标识

a=extmap:14 urn:ietf:params:rtp-hdrext:toffset
- **RTP头扩展**：
  - ID：`14`
  - URI：`toffset`，传输时间偏移

a=extmap:2 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
- **RTP头扩展**：ID `2`，绝对发送时间

a=extmap:13 urn:3gpp:video-orientation
- **RTP头扩展**：
  - ID：`13`
  - URI：`video-orientation`，视频方向信息

a=extmap:3 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01
- **RTP头扩展**：ID `3`，传输层拥塞控制

a=extmap:5 http://www.webrtc.org/experiments/rtp-hdrext/playout-delay
- **RTP头扩展**：
  - ID：`5`
  - URI：`playout-delay`，播放延迟建议

a=extmap:6 http://www.webrtc.org/experiments/rtp-hdrext/video-content-type
- **RTP头扩展**：
  - ID：`6`
  - URI：`video-content-type`，视频内容类型

a=extmap:7 http://www.webrtc.org/experiments/rtp-hdrext/video-timing
- **RTP头扩展**：
  - ID：`7`
  - URI：`video-timing`，视频定时信息

a=extmap:8 http://www.webrtc.org/experiments/rtp-hdrext/color-space
- **RTP头扩展**：
  - ID：`8`
  - URI：`color-space`，视频色彩空间信息

a=extmap:4 urn:ietf:params:rtp-hdrext:sdes:mid
- **RTP头扩展**：ID `4`，媒体标识（MID）

a=extmap:10 urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id
- **RTP头扩展**：
  - ID：`10`
  - URI：`rtp-stream-id`，RTP流标识

a=extmap:11 urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id
- **RTP头扩展**：
  - ID：`11`
  - URI：`repaired-rtp-stream-id`，修复的RTP流标识

a=sendonly
- **发送方向**：只发送，不接收

a=msid:FAMS FAMSv1
- **媒体流标识**：
  - 流ID：`FAMS`
  - 轨道ID：`FAMSv1`

a=rtcp-mux
- **RTCP复用**：将RTCP与RTP复用同一端口

a=rtcp-rsize
- **RTCP缩减大小**：使用较小的RTCP报文（RFC 5506）

a=rtpmap:96 H264/90000
- **RTP映射**：
  - 编码：`H264`
  - 采样率：`90000`Hz

a=rtcp-fb:96 goog-remb
- **RTCP反馈**：
  - 针对编码`96`，支持`goog-remb`带宽估计

a=rtcp-fb:96 transport-cc
- **RTCP反馈**：
  - 支持传输层拥塞控制

a=rtcp-fb:96 ccm fir
- **RTCP反馈**：
  - 支持`ccm fir`（全帧请求）

a=rtcp-fb:96 nack
- **RTCP反馈**：
  - 支持`nack`（否定确认）用于丢包重传

a=rtcp-fb:96 nack pli
- **RTCP反馈**：
  - 支持`pli`（图片丢失指示），请求关键帧

a=fmtp:96 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=640c34
- **格式参数**：
  - 允许非对称级别：`level-asymmetry-allowed=1`
  - 分包模式：`packetization-mode=1`（非交错模式）
  - 配置文件级别：`profile-level-id=640c34`（High Profile，Level 3.4）

a=rtpmap:97 rtx/90000
- **RTP映射**：
  - 编码：`rtx`（重传）
  - 采样率：`90000`Hz

a=fmtp:97 apt=96
- **格式参数**：
  - 关联有效负载类型：`apt=96`（对应H264编码）

a=rtpmap:98 H264/90000
- **RTP映射**：
  - 编码：`H264`
  - 采样率：`90000`Hz

a=rtcp-fb:98 goog-remb
- **RTCP反馈**：同编码`96`

a=rtcp-fb:98 transport-cc
- **RTCP反馈**：同编码`96`

a=rtcp-fb:98 ccm fir
- **RTCP反馈**：同编码`96`

a=rtcp-fb:98 nack
- **RTCP反馈**：同编码`96`

a=rtcp-fb:98 nack pli
- **RTCP反馈**：同编码`96`

a=fmtp:98 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42e034
- **格式参数**：
  - 配置文件级别：`profile-level-id=42e034`（Baseline Profile，Level 3.4）

a=rtpmap:99 rtx/90000
- **RTP映射**：
  - 编码：`rtx`
  - 采样率：`90000`Hz

a=fmtp:99 apt=98
- **格式参数**：
  - 关联有效负载类型：`apt=98`（对应H264编码）

a=rtpmap:100 red/90000
- **RTP映射**：
  - 编码：`red`（冗余编码）
  - 采样率：`90000`Hz

a=rtpmap:101 rtx/90000
- **RTP映射**：
  - 编码：`rtx`
  - 采样率：`90000`Hz

a=fmtp:101 apt=100
- **格式参数**：
  - 关联有效负载类型：`apt=100`（对应red编码）

a=rtpmap:127 ulpfec/90000
- **RTP映射**：
  - 编码：`ulpfec`（前向纠错）
  - 采样率：`90000`Hz

a=ssrc-group:FID 439044504 650966991
- **SSRC组**：
  - 组语义：`FID`（流标识）
  - SSRC列表：`439044504`（主流），`650966991`（重传流）

a=ssrc:439044504 cname:cXfVAjx41LASIlBT
- **同步源（SSRC）标识**：
  - SSRC：`439044504`
  - CNAME：`cXfVAjx41LASIlBT`

a=ssrc:439044504 msid:FAMS FAMSv1
- **媒体流标识**：
  - 流ID：`FAMS`
  - 轨道ID：`FAMSv1`

a=ssrc:650966991 cname:cXfVAjx41LASIlBT
- **同步源（SSRC）标识**：
  - SSRC：`650966991`
  - CNAME：`cXfVAjx41LASIlBT`

a=ssrc:650966991 msid:FAMS FAMSv1
- **媒体流标识**：
  - 流ID：`FAMS`
  - 轨道ID：`FAMSv1`

//==数据通道描述==

m=application 9 UDP/DTLS/SCTP webrtc-datachannel
- **媒体描述**：
  - 媒体类型：`application`（应用数据）
  - 端口：`9`（通过ICE协商）
  - 传输协议：`UDP/DTLS/SCTP`（基于DTLS的SCTP传输）
  - 格式：`webrtc-datachannel`，表示WebRTC数据通道

c=IN IP4 0.0.0.0
- **连接信息**：与前面一致

a=ice-ufrag:2D6J
- **ICE用户名碎片**：`2D6J`，与前面一致

a=ice-pwd:jWsbseLplx44YJwXRravag7Y
- **ICE密码**：`jWsbseLplx44YJwXRravag7Y`，与前面一致

a=ice-options:trickle renomination
- **ICE选项**：支持Trickle ICE和重新提名

a=fingerprint:sha-256 F4:57:8B:7E:6C:A1:0F:08:95:72:67:68:3C:18:67:40:39:D9:D5:0A:A4:6E:C0:3E:67:BA:E0:02:B5:FF:EF:40
- **DTLS指纹**：用于安全验证

a=setup:actpass
- **DTLS连接角色**：双方均可主动或被动

a=mid:2
- **媒体标识符**：`2`，用于BUNDLE中的媒体流标识

a=sctp-port:5000
- **SCTP端口**：`5000`，用于SCTP协议的传输

a=max-message-size:262144
- **最大消息大小**：`262144`字节，数据通道单次消息的最大尺寸
```

## SDP - Candidate
candidate: 一个可能的网络连接点的候选项。**网络节点负责最终的音视频流传输的数据通道，很重要**。
sdpMid/sdpMLineIndex: 用来标记 SDP 中的媒体流索引。即每一个 candidate 候选，都属于某一个媒体流通道，如 音频流、视频流等。
```
{
  "candidate": "candidate:4109260943 1 udp 8331263 x.x.x.x 36634 typ relay raddr x.x.x.x rport 63859 generation 0 ufrag 2D6J network-id 1 network-cost 10",
  "sdpMLineIndex": 0,
  "sdpMid": "0"
}

优先级排序：
1. 候选类型：host > srflx > relay
2. 网络成本 network-cost：wifi > 蜂窝
```

下面示例，是使用 Cloudflare TURN 服务采集到的一个网络发现节点：
**candidate:4109260943 1 udp 8331263 x.x.x.x 36634 typ relay raddr x.x.x.x rport 63859 generation 0 ufrag 2D6J network-id 1 network-cost 10**
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091629139.png)
tips:
* TCP or UDP：webrtc 使用 SRTP 来传输音视频流，该应用层协议可同时在 TCP 和 UDP 上工作。其中，默认使用 UDP。只有 UDP 不可用后会回退到 TCP。
* N 个 candidate 都是候选并被 webrtc sdk 存储。当 webrtc 使用过程中【中断】了，会自行寻找其他的 candidate 作为候选，对开发者透明（自动重连）。

### Candidate Raw Example:
```
[
  {
    "candidate": "candidate:2524315334 1 udp 2122260223 10.0.3.39 59309 typ host generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:2489915706 1 udp 2122194687 169.254.125.209 51656 typ host generation 0 ufrag 2D6J network-id 2 network-cost 10",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:1463068104 1 udp 2121998079 192.0.0.1 54311 typ host generation 0 ufrag 2D6J network-id 7 network-cost 900",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:878085806 1 udp 2122131711 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 50281 typ host generation 0 ufrag 2D6J network-id 8 network-cost 900",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:3258950406 1 udp 2122066175 240b:c010:823:5c21:40a5:9a43:ff0c:906 58647 typ host generation 0 ufrag 2D6J network-id 9 network-cost 900",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:306562041 1 udp 2121935103 240b:c010:823:5c21:18b6:b94e:abe4:cec2 53502 typ host generation 0 ufrag 2D6J network-id 3 network-cost 50",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:306562041 1 udp 2121869567 240b:c010:823:5c21:18b6:b94e:abe4:cec2 54246 typ host generation 0 ufrag 2D6J network-id 4 network-cost 50",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:2524315334 1 udp 2122260223 10.0.3.39 49555 typ host generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:2489915706 1 udp 2122194687 169.254.125.209 50166 typ host generation 0 ufrag 2D6J network-id 2 network-cost 10",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:1463068104 1 udp 2121998079 192.0.0.1 53998 typ host generation 0 ufrag 2D6J network-id 7 network-cost 900",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:878085806 1 udp 2122131711 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 52161 typ host generation 0 ufrag 2D6J network-id 8 network-cost 900",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:3258950406 1 udp 2122066175 240b:c010:823:5c21:40a5:9a43:ff0c:906 60559 typ host generation 0 ufrag 2D6J network-id 9 network-cost 900",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:306562041 1 udp 2121935103 240b:c010:823:5c21:18b6:b94e:abe4:cec2 50196 typ host generation 0 ufrag 2D6J network-id 3 network-cost 50",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:3975092077 1 tcp 1517955327 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63867 typ host tcptype passive generation 0 ufrag 2D6J network-id 3 network-cost 50",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:3975092077 1 tcp 1517889791 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63868 typ host tcptype passive generation 0 ufrag 2D6J network-id 4 network-cost 50",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:1759455826 1 tcp 1518280447 10.0.3.39 63862 typ host tcptype passive generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:1791217070 1 tcp 1518214911 169.254.125.209 63863 typ host tcptype passive generation 0 ufrag 2D6J network-id 2 network-cost 10",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:2845733212 1 tcp 1518018303 192.0.0.1 63864 typ host tcptype passive generation 0 ufrag 2D6J network-id 7 network-cost 900",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:3405533754 1 tcp 1518151935 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 63865 typ host tcptype passive generation 0 ufrag 2D6J network-id 8 network-cost 900",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:1016428434 1 tcp 1518086399 240b:c010:823:5c21:40a5:9a43:ff0c:906 63866 typ host tcptype passive generation 0 ufrag 2D6J network-id 9 network-cost 900",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:1759455826 1 tcp 1518280447 10.0.3.39 63869 typ host tcptype passive generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:1791217070 1 tcp 1518214911 169.254.125.209 63870 typ host tcptype passive generation 0 ufrag 2D6J network-id 2 network-cost 10",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:2845733212 1 tcp 1518018303 192.0.0.1 63871 typ host tcptype passive generation 0 ufrag 2D6J network-id 7 network-cost 900",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:3405533754 1 tcp 1518151935 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 63872 typ host tcptype passive generation 0 ufrag 2D6J network-id 8 network-cost 900",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:1016428434 1 tcp 1518086399 240b:c010:823:5c21:40a5:9a43:ff0c:906 63873 typ host tcptype passive generation 0 ufrag 2D6J network-id 9 network-cost 900",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:3975092077 1 tcp 1517955327 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63874 typ host tcptype passive generation 0 ufrag 2D6J network-id 3 network-cost 50",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:1759455826 1 tcp 1518280447 10.0.3.39 63876 typ host tcptype passive generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:1791217070 1 tcp 1518214911 169.254.125.209 63877 typ host tcptype passive generation 0 ufrag 2D6J network-id 2 network-cost 10",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:2845733212 1 tcp 1518018303 192.0.0.1 63878 typ host tcptype passive generation 0 ufrag 2D6J network-id 7 network-cost 900",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:3405533754 1 tcp 1518151935 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 63879 typ host tcptype passive generation 0 ufrag 2D6J network-id 8 network-cost 900",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:1016428434 1 tcp 1518086399 240b:c010:823:5c21:40a5:9a43:ff0c:906 63880 typ host tcptype passive generation 0 ufrag 2D6J network-id 9 network-cost 900",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:3975092077 1 tcp 1517955327 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63881 typ host tcptype passive generation 0 ufrag 2D6J network-id 3 network-cost 50",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:3975092077 1 tcp 1517889791 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63882 typ host tcptype passive generation 0 ufrag 2D6J network-id 4 network-cost 50",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:1410854543 1 udp 25108735 x.x.x.x 53914 typ relay raddr x.x.x.x rport 63839 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:1101771793 1 udp 41886207 x.x.x.x 53010 typ relay raddr x.x.x.x rport 49555 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:1101771793 1 udp 41886207 x.x.x.x 52333 typ relay raddr x.x.x.x rport 59309 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:1541054091 1 udp 25108735 x.x.x.x 52668 typ relay raddr x.x.x.x rport 63829 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  },
  {
    "candidate": "candidate:3011387645 1 udp 25108735 x.x.x.x 44090 typ relay raddr x.x.x.x rport 63857 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:351976030 1 udp 8331263 x.x.x.x 46887 typ relay raddr x.x.x.x rport 63834 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 1,
    "sdpMid": "1"
  },
  {
    "candidate": "candidate:4109260943 1 udp 8331263 x.x.x.x 36634 typ relay raddr x.x.x.x rport 63859 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 2,
    "sdpMid": "2"
  },
  {
    "candidate": "candidate:2106314174 1 udp 8331263 x.x.x.x 23955 typ relay raddr x.x.x.x rport 63831 generation 0 ufrag 2D6J network-id 1 network-cost 10",
    "sdpMLineIndex": 0,
    "sdpMid": "0"
  }
]
```

### Candidate lines:
```
candidate:2524315334 1 udp 2122260223 10.0.3.39 59309 typ host generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:2489915706 1 udp 2122194687 169.254.125.209 51656 typ host generation 0 ufrag 2D6J network-id 2 network-cost 10
candidate:1463068104 1 udp 2121998079 192.0.0.1 54311 typ host generation 0 ufrag 2D6J network-id 7 network-cost 900
candidate:878085806 1 udp 2122131711 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 50281 typ host generation 0 ufrag 2D6J network-id 8 network-cost 900
candidate:3258950406 1 udp 2122066175 240b:c010:823:5c21:40a5:9a43:ff0c:906 58647 typ host generation 0 ufrag 2D6J network-id 9 network-cost 900
candidate:306562041 1 udp 2121935103 240b:c010:823:5c21:18b6:b94e:abe4:cec2 53502 typ host generation 0 ufrag 2D6J network-id 3 network-cost 50
candidate:306562041 1 udp 2121869567 240b:c010:823:5c21:18b6:b94e:abe4:cec2 54246 typ host generation 0 ufrag 2D6J network-id 4 network-cost 50
candidate:2524315334 1 udp 2122260223 10.0.3.39 49555 typ host generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:2489915706 1 udp 2122194687 169.254.125.209 50166 typ host generation 0 ufrag 2D6J network-id 2 network-cost 10
candidate:1463068104 1 udp 2121998079 192.0.0.1 53998 typ host generation 0 ufrag 2D6J network-id 7 network-cost 900
candidate:878085806 1 udp 2122131711 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 52161 typ host generation 0 ufrag 2D6J network-id 8 network-cost 900
candidate:3258950406 1 udp 2122066175 240b:c010:823:5c21:40a5:9a43:ff0c:906 60559 typ host generation 0 ufrag 2D6J network-id 9 network-cost 900
candidate:306562041 1 udp 2121935103 240b:c010:823:5c21:18b6:b94e:abe4:cec2 50196 typ host generation 0 ufrag 2D6J network-id 3 network-cost 50
candidate:3975092077 1 tcp 1517955327 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63867 typ host tcptype passive generation 0 ufrag 2D6J network-id 3 network-cost 50
candidate:3975092077 1 tcp 1517889791 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63868 typ host tcptype passive generation 0 ufrag 2D6J network-id 4 network-cost 50
candidate:1759455826 1 tcp 1518280447 10.0.3.39 63862 typ host tcptype passive generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:1791217070 1 tcp 1518214911 169.254.125.209 63863 typ host tcptype passive generation 0 ufrag 2D6J network-id 2 network-cost 10
candidate:2845733212 1 tcp 1518018303 192.0.0.1 63864 typ host tcptype passive generation 0 ufrag 2D6J network-id 7 network-cost 900
candidate:3405533754 1 tcp 1518151935 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 63865 typ host tcptype passive generation 0 ufrag 2D6J network-id 8 network-cost 900
candidate:1016428434 1 tcp 1518086399 240b:c010:823:5c21:40a5:9a43:ff0c:906 63866 typ host tcptype passive generation 0 ufrag 2D6J network-id 9 network-cost 900
candidate:1759455826 1 tcp 1518280447 10.0.3.39 63869 typ host tcptype passive generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:1791217070 1 tcp 1518214911 169.254.125.209 63870 typ host tcptype passive generation 0 ufrag 2D6J network-id 2 network-cost 10
candidate:2845733212 1 tcp 1518018303 192.0.0.1 63871 typ host tcptype passive generation 0 ufrag 2D6J network-id 7 network-cost 900
candidate:3405533754 1 tcp 1518151935 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 63872 typ host tcptype passive generation 0 ufrag 2D6J network-id 8 network-cost 900
candidate:1016428434 1 tcp 1518086399 240b:c010:823:5c21:40a5:9a43:ff0c:906 63873 typ host tcptype passive generation 0 ufrag 2D6J network-id 9 network-cost 900
candidate:3975092077 1 tcp 1517955327 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63874 typ host tcptype passive generation 0 ufrag 2D6J network-id 3 network-cost 50
candidate:1759455826 1 tcp 1518280447 10.0.3.39 63876 typ host tcptype passive generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:1791217070 1 tcp 1518214911 169.254.125.209 63877 typ host tcptype passive generation 0 ufrag 2D6J network-id 2 network-cost 10
candidate:2845733212 1 tcp 1518018303 192.0.0.1 63878 typ host tcptype passive generation 0 ufrag 2D6J network-id 7 network-cost 900
candidate:3405533754 1 tcp 1518151935 240b:c010:401:2c70:518d:9e8c:aa43:2dc4 63879 typ host tcptype passive generation 0 ufrag 2D6J network-id 8 network-cost 900
candidate:1016428434 1 tcp 1518086399 240b:c010:823:5c21:40a5:9a43:ff0c:906 63880 typ host tcptype passive generation 0 ufrag 2D6J network-id 9 network-cost 900
candidate:3975092077 1 tcp 1517955327 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63881 typ host tcptype passive generation 0 ufrag 2D6J network-id 3 network-cost 50
candidate:3975092077 1 tcp 1517889791 240b:c010:823:5c21:18b6:b94e:abe4:cec2 63882 typ host tcptype passive generation 0 ufrag 2D6J network-id 4 network-cost 50
candidate:1410854543 1 udp 25108735 x.x.x.x 53914 typ relay raddr x.x.x.x rport 63839 generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:1101771793 1 udp 41886207 x.x.x.x 53010 typ relay raddr x.x.x.x rport 49555 generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:1101771793 1 udp 41886207 x.x.x.x 52333 typ relay raddr x.x.x.x rport 59309 generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:1541054091 1 udp 25108735 x.x.x.x 52668 typ relay raddr x.x.x.x rport 63829 generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:3011387645 1 udp 25108735 x.x.x.x 44090 typ relay raddr x.x.x.x rport 63857 generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:351976030 1 udp 8331263 x.x.x.x 46887 typ relay raddr x.x.x.x rport 63834 generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:4109260943 1 udp 8331263 x.x.x.x 36634 typ relay raddr x.x.x.x rport 63859 generation 0 ufrag 2D6J network-id 1 network-cost 10
candidate:2106314174 1 udp 8331263 x.x.x.x 23955 typ relay raddr x.x.x.x rport 63831 generation 0 ufrag 2D6J network-id 1 network-cost 10,
```

## 网络发现 - STUN
以上媒体协商的过程中，SDP 的很多信息都可以由开发者控制，比如编码类型。但是 candidate 网络节点的收集会比较复杂。
数据包如何进入公网
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091631997.png)

NAT 路由映射表：
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091632870.png)

### NAT 的多样性
路由器具有特定的类型：**完全锥形**、**地址限制锥形**、**端口限制锥形**、**对称NAT**
映射表规则：
1. 映射端口：
  * 对称 NAT → {source_ip, source_port, dest_ip, dest_port} 有一个变化，将新建映射端口
  * 端口限制锥形 / 地址限制锥形 → {source_ip, source_port} 有一个变化，将新建映射端口
2. 回流数据包：
  * 对称 NAT / 端口限制锥形：被【映射表】映射过的 {dest_ip, dest_port} 可以回流
  * 地址限制锥形：被【映射表】映射过的 {dest_ip} 可以回流
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091633623.png)

### P2P 打洞
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091633216.png)
对于 p2p 打洞失败的场景，就需要 TURN 做中继服务，转发音视频流。

## 中继服务 - TURN
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091635057.png)

## 协商回退
在 ipv6 越多的场景下，上面提到的 ipv4 NAT 问题可以有效解决。但 STUN 网络发现依旧不可缺少。local、多 ipv6、防火墙等场景依旧需要网络择优。
ipv6 场景下，不需要打洞就能直连，最具有性价比。以下场景可以专门做【协商回退】处理：
1. server 端 ipv6 和 ipv4 共存，那么 client 端只要有 ipv6 就明确使用 ipv6
```
pc.onicecandidate = event => {
  if (event.candidate) {
    if (event.candidate.address.indexOf(':') !== -1) {
      // IPv6候选者，可能优先处理
      handleIPv6Candidate(event.candidate);
    }
  }
};
```
2. 回退（ipv6 → ipv4 → turn 中继）
```
pc.oniceconnectionstatechange = () => {
  switch(this.pc.iceConnectionState) {
    case 'checking':
      // 设置连接超时
      connectionTimer = setTimeout(() => {
        this.handleConnectionTimeout();
      }, this.connectionTimeout);
      break;
  }
};

async handleConnectionTimeout() {
  console.log('连接超时，尝试回退策略');
  if (this.preferredProtocol === 'IPv6') {
    console.log('切换到IPv4');
    this.preferredProtocol = 'IPv4';
    await this.restartConnection();
  } else {
    console.log('尝试TURN服务器');
    await this.switchToTURN();
  }
}
```
# 信令服务的演变
1. 自建 IM - socket / websocket。
  * 最复杂，最可控。可以把媒体协商、会控等一系列功能做体系化的整合。
  * 私有独立服务使用该场景较多，如某个独立的 app 等。
2. SSE 长链接推送 / HTTP 轮询，
  * [SSE 指南](https://www.yigegongjiang.com/2024/sse/)
3. firebase realtime database - db + im
  * [RTDB 指南](https://firebase.google.com/docs/database?hl=zh-cn)
4. WHIP & WHEP：正在逐渐形成标准，普遍适用于各大推流平台。（适用于中央服务器，因为服务器具有固定的 sdp - candidate）
  * 通过 restful api 控制媒体协商和会话生命周期
  * 可以通过 header、body 增加授权、编码定义、期望格式和分辨率等等参数
  * 平台场景使用较多，如直播平台、云厂商。
```
Example:
https://webrtcpush.tlivewebrtcpush.com/webrtc/v2/whip

// WHIP 服务
const server = http.createServer((req, res) => {
  // 处理 WHIP 客户端请求
  switch (req.method) {
    case "POST":
      // 生成 SDP Answer
      const sdpAnswer = generateSdpAnswer(sdpOffer);
      res.statusCode = 201;
      res.setHeader("Location", `/whip/sessions/${randomUUID()}`);// 用于 delete
      res.end(sdpAnswer);
    case "PATCH":
      // 处理 ICE 候选
      let iceCandidate = req.text();
      handleIceCandidate(iceCandidate);
    case "DELETE":
      // 处理会话删除
      deleteSession(req.sessionId);
    case "OPTIONS":
      // 处理能力协商
      res.setHeader("Allow", "POST, PATCH, DELETE, OPTIONS");
      res.setHeader("Accept", "application/sdp");
  }
  res.end();
});

function generateSdpAnswer(sdpOffer) {
  return `v=0 o=- 123456789 123456789 IN IP4 0.0.0.0 s=- t=0 0 m=video 9 UDP/TLS/RTP/SAVPF 96 c=IN IP4 0.0.0.0 a=rtpmap:96 VP8/90000 a=setup:active a=mid:0 `;
}

function handleIceCandidate(iceCandidate) {
  console.log("Received ICE candidate:", iceCandidate);
}

function deleteSession(sessionId) {
  console.log("Deleting session:", sessionId);
}
```
# RTC DataChannel
当 webrtc 的两个端媒体协商完成后，可通过 webrtc 在两个端创建 全双工 的通信通道，即 RTC Data Channel。
应用场景：
1. 聊天、简单的会控
2. 会议白版、云游戏
3. 飞书完成了高效的协同编辑 - 妙享 Magic Share
  * 会议中，A 进行文档共享（非投屏）
  * 其他人员默认跟随 A 的操作进行文档阅读（根队）
  * 其他人员可自行操作、编辑文档（离队）
  * 核心算法还是云文档协作，但通过 DataChannel 非常高效的完成会议结果的沉淀。

# RTC SFU - MCU 架构
在不同的参会规模场景下，有多种流传输方案。
![](https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202411091640934.png)

P2P 打洞场景最特殊，因为是终端直连，服务器无法监控通话质量。这种场景也最安全，视频流不会被服务器侧拦截和分析。
  * telegram 在 webrtc 自身的 dtls 流加密基础上，增加了 MTProto 端到端 加密。可以避免被中间服务分析流数据。
  
MCU 方案，server 需要对大量的【视频流、音频流】进行合流，增加了延时。可控性也最高，可以做各种场景的滤镜、变声、流组合特效等。
  * mcu 方案，不支持端到端加密。
  
现实场景下，除了视频聊天这种 1-1 场景对 webrtc 是强依赖外，其他流媒体服务如直播，都是多协议共存。
  * 推流：webrtc、rtmp、基于 webrtc 的私有协议(优化延迟)
  * 拉流：webrtc、hls、rtmp、flv
  * 辅助：直播拉流场景，CDN 必不可少
  
# WebRTC 源码宝藏
WebRTC 中 3A(AEC、AGC、ANC) 音频算法最顶级处理音频的算法，可以直接拿来用。
对于网络方向，网络带宽的评估、平滑处理、网络协议的实现在 WebRTC 中是应有尽有。

___


