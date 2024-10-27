---
title: 1.1.1.1、WARP 和 MASQUE
date: 2024-10-27 16:29:36
categories:
- 技术
tags:
- 网络
---

> 实际测试结果，虽然 Cloudflare 号称服务部署非常多，但使用 Warp 的 【VPN/代理】功能，会明显增加网络延迟。（location & ip： Japan）

Cloudflare 提供了 1.1.1.1 DNS 服务，以及 WARP VPN + 代理。

# 公共 DNS

## 运营商 DNS 有什么问题？

直接使用运营商的 DNS 服务，有下面两个重要的问题：
1. ISP 会通过 DNS 查询的域名，分析用户访问的网站。（DNS 查询是明文的 UDP）
2. ISP DNS 会有劫持、污染、缓存等

那么公共 DNS 可以解决上面的问题吗？**不行**。

对于普通用户来说，使用运营商 DNS 没有任何问题，也会加快域名解析的速度。
如果想要安全，避免被运营商做【DNS 查询】分析，公共 DNS 无能为力。不管使用的公共 DNS 是 ipv4、ipv6、DoH，都无法解决。
DoH 是能用到最安全的级别了，但还是有 SNI 泄漏。

<!-- more -->

```
- IPv4 和 IPv6：
  - 1.1.1.1 和 2606:4700:4700::1111
  - 1.0.0.1 和 2606:4700:4700::1001
- DoH：
  - https://cloudflare-dns.com/dns-query
```

## 公共 DNS 如何实现负载均衡

通过域名，可以在 DNS 权威服务器上做 IP 的负载均衡。
那么，对于只有 IP 地址而没有域名的 DNS 服务，如何实现负载均衡呢？

每个 IP 能被发现，是通过网络节点之间的 BGP（边界网关协议）进行广播。
通过 **Anycast** 技术，可以在不同地理位置的多台主机上部署相同的 IP，从而让客户端就近连接到 IP 服务器。

# WARP

WARP 是 Cloudflare 提供的一种 VPN 服务，使用了 WireGuard 协议。
不同的 VPN 协议分别工作在网络模型的不同层次，其中 WireGuard 工作在第三层（网络层）。
因为 iOS 等系统，同一时间只能开启一个 VPN 服务，所以使用 WARP 的时候，数据包流向为：
1. WARP app 拦截所有三层数据包并转发到 WARP 的 WireGuard 服务器，服务器向具体的 domain 发起请求。

在 Mac 系统上，除了 WARP 还可以手动设置网络代理（Thrid Proxys），这些网络代理一般工作在应用层。这个时候数据包流向为：
1. 数据包转到 Third Proxys 进行封装，目标 domain 变成【代理 服务器】
2. WARP app 拦截所有三层数据包转发到【WARP 服务器】
3. 【WARP 服务器】向【代理 服务器】发起请求
4. 【代理 服务器】向具体的 domain 发起请求

# MASQUE

> 通常使用的翻墙工具并不是 VPN，而是网络代理（更准确地说，称为“正向代理”）。
> 之所以经常将它们称为 VPN，是因为在 iOS 等设备上，需要开启 VPN 服务，这些软件才能拦截数据流量，实现代理转发。
> 但它们与通常所指的 WireGuard、OpenVPN 等 VPN 协议并不一致。


WARP 推出了基于 MASQUE 协议的版本，用于替代 WireGuard。MASQUE 使用了 QUIC 协议，属于应用层的网络数据转发。

从本质上来说，此时的 WARP 已经不能称为 VPN，而是“正向代理”。具体来说：

- WARP 在网络分层的底层（第三层网络层）捕获系统流量，这一点与使用 WireGuard 时相同。不同的是后续的处理方式：
  - **WireGuard**：使用该协议时，采用的是 VPN 方案。对拦截的数据包进行加密，然后通过隧道发送。
  - **MASQUE**：使用该协议时，采用的是正向代理方案。对拦截的数据包，通过 QUIC 协议代理到 WARP 服务端节点。

这种技术方案与 macOS 上著名的代理软件 Surge 非常相似。Surge 有一个功能叫“增强模式”。

- 在正常情况下，Surge 通过在系统网络设置中的代理配置设置 localhost 代理，捕获应用层的流量进行转发。
- 开启增强模式后，Surge 会通过第三层捕获所有流量包，接管后进行转发。

WARP 为什么抛弃【WireGuard】使用 【MASQUE】，可以参考官方介绍：
[https://blog.cloudflare.com/masque-building-a-new-protocol-into-cloudflare-warp/](https://blog.cloudflare.com/masque-building-a-new-protocol-into-cloudflare-warp/)


___


