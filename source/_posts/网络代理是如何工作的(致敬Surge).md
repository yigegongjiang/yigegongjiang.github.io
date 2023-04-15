---
title: 网络代理是如何工作的(致敬Surge)
categories:
- 技术
tags:
- 计算机基础
- 网络
keys: 
---

最近本网站的 HTTPS SSL 证书过期了，于是去域名管理平台重新申请了一下。
无意中通过 dig 发现，网站的 DNS 解析 IP 一直是 **198.18.1.xxx** 这些。因为我的域名是 CNAME 映射到 github 的，所以有 dig 了一下 github 对应的 ip，发现也是 **198.18.1.xxx**。
从哪个角度来看，至少都有些问题。我用的 DNSPod 域名解析平台，dig 自定义域名和 github page 域名，怎么也不能在同一个网段里。
搜索了一下才知道，原来 ip 198 不是公网 ip，之前我以为内网 ip 是 10/192 这些，知识还是有局限。
> 198.18.0.0/15  198.18.0.0 – 198.19.255.255  131,072  专用网络  用于测试两个不同的子网的网间通信。
> https://zh.wikipedia.org/wiki/保留IP地址

在家庭网络中 dig 互联网的域名，为什么 dns 解析成了 198.x 呢？最后发现是我最常使用的网络软件 **Surge** 引起的。
**Surge** 这些年给了我很多帮助，很感谢。特意开此文，讲解 Surge 工作原理，以致敬 Surge。
本文会对 Socket、Wireshark、网络系统代理(http/s、socket5、POSIX)、网络网卡代理(VIF)、VPN、DNS、DOH(SNI) 等知识点进行描述，以更加全面的讲解 Surge 的工作原理。

# 致敬 Surge 

Surge 是一款非常强大的网络调试工具，很多人都对它极为陌生，主要原因是它的售价过高，宣传和使用的人也不多。
我认为每一个 ITer 都应该使用它。网络在 IT 工作中时刻都需要关注，如 DNS 解析、网络流量查看和 hook、网络代理等等，这些 Surge 都可以做到。是发现和排查网络问题的神器，也可以协助工作。
这里我会先介绍下 Surge 的使用，或许你会感兴趣。如果以后 Surge 帮助到了你，那也是一件幸事。

<!-- more -->

Surge 只有 iOS 和 Mac 版，是订阅制，我简单介绍下订阅制，如下图：
![](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202304160218073.png)
Surge iOS 和 Mac 需要单独购买，iOS 只能在非国区 apple store 下载，Mac 可以在[官网](https://nssurge.com)下载。
价格是有些贵，iOS 和 Mac 差不多都要 $49.9。Mac 版建议拼车，140 元左右，iOS 不建议拼车，比较麻烦。两个平台都可以试用下，效果不错再订阅。
这里有 Surge 的答疑，如：[Mac release notes](https://kb.nssurge.com/surge-knowledge-base/v/zh/release-notes/surge-mac-5)、[iOS release notes](https://kb.nssurge.com/surge-knowledge-base/v/zh/release-notes/surge-ios)、[常见问题](https://kb.nssurge.com/surge-knowledge-base/v/zh/license/pre-sale)、[订阅说明](https://kb.nssurge.com/surge-knowledge-base/v/zh/license/ios-fus)。
下面是一些 Surge 的常见使用：
**http/https/socks5代理**、**机场流量转发**(懂的)、**软路由**(相当强悍)、**MitM**&readWrite(比金瓶梅Charles好用)、**DNS代理**、**模块和脚本**(强大功能，非专业人士不常用)。
iOS 和 Mac 可以互联，通过 Mac 连上 iOS 后，可以快速设置&查看 iOS 上的网络流量。
如果对于刚需人群，软路由功能就已经让 Surge 的售价低到尘埃里了，懂得已懂。

下面对 Surge 是如何做网络流量劫持代理转发等核心功能，做技术分析。

# 基石

## Socket

在之前的[IM 和 Socket 的关系及 Heart 的必要性](https://www.yigegongjiang.com/2020/IM%E5%92%8CSocket%E7%9A%84%E5%85%B3%E7%B3%BB%E5%8F%8AHeart%E7%9A%84%E5%BF%85%E8%A6%81%E6%80%A7/)文章中，对 Socket 套接字和 Socket库 有说明。
一定要理解 **gethostbyname()**、**socket()**、**bind()**、**connect()**、**listen()**、**accept()**、**send()/recv()和write()/read()** 这些 socket api，否则无法理解 tcp/udp 的数据包传输，也无法理解下面要说的网络代理。

## Wireshark

在之前的[TCP 数据传输过程分析](https://www.yigegongjiang.com/2020/TCP%20%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93%E8%BF%87%E7%A8%8B%E5%88%86%E6%9E%90/)文章中，有对 Wireshark 如何抓包和数据分析进行过阐述。Wireshark 是网络流量分析的神器，在理解 surge 的过程中，我对 tls、sni、dns、vif、lookback 等场景都进行了抓包分析和验证，非常有帮助。

# 网络代理-系统代理

## http

## https

## socks5

## 终端默认为什么不走代理

### POSIX 是什么

解释 Posix 标准
C 标准库/运行时库
glibc / libc / MCRT(msvcrt)

POSIX线程 - https://zh.wikipedia.org/wiki/POSIX%E7%BA%BF%E7%A8%8B
\#include <pthread.h>
\#include <threads.h>

pthread和thread都是多线程编程的相关库。
pthread是POSIX标准中的线程库，它在许多操作系统上都得到了实现，包括Unix、Linux、macOS等。pthread提供了一组高级的线程函数，使得多线程编程更加方便。
thread是C++11标准中的线程库，它主要应用于C++开发，并且仅在支持C++11标准的操作系统上才能使用。thread提供了类似于STL的接口，使得多线程编程更加高效。
总的来说，pthread和thread的区别在于：pthread是POSIX标准中的线程库，主要适用于跨平台编程；thread是C++11标准中的线程库，主要适用于C++开发。

# 网络代理-虚拟网卡

# VPN&代理差异

# DNS

域名解析运营商的配置后台
dns 的递归和迭代查询

gethostbyname()
// 引出 dns over https

# DOH(httpdns)
记的有一次问过一个候选人问题，就是对于一个网络请求，是 TCP 三次握手先发生，还是 HTTPS 安全认证先发生。
对于 DOH，会引申一个问题：正式的 https 网络请求发出前，需要先做 doh 请求，拿到 网络域名对应的 ip。而 doh 也是一个 https 请求，本身也需要做 ssl/tls 鉴权。

dns over https：如何更高效的 dns - 移动端方案（hook 请求 / hook getipbyname）

surge 开放的 网络知识 ： SNI等

okhttp/afnetwork 怎么进行网络请求，以及安全验证
面向信仰编程系列 - https://draveness.me/afnetworking5/

DNS 什么时候做解析，什么时候不做解析

由浅入深写代理-系列
https://www.zhihu.com/people/facert/posts?page=2


Android 网络优化，使用 HTTPDNS 优化 DNS，从原理到 OkHttp 集成
https://juejin.cn/post/6844903806216617992
Android端HTTPS（含SNI）业务场景：IP直连方案
https://www.alibabacloud.com/help/zh/httpdns/latest/connect-an-android-app-to-an-ip-address-over-https
iOS端HTTPS（含SNI）业务场景“IP直连”方案说明
https://help.aliyun.com/document_detail/271786.html
