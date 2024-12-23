---
title: 老梗-浏览器按回车后发生了什么
date: 2022-10-23 02:05:14
categories:
- 技术
tags:
- 网络
- 计算机原理
---

想起了一个老问题：浏览器按下回车的时候，后续流程是怎么变化的。
这个问题非常精妙，基本上把网络问题一次打包了。
对相关知识面了解越多的人，能说的内容也越多。越资深的人，能说的时间也越长。
我对网络也一直感兴趣，索性这次就做了大图，越做就盘子越大耗时越多，每个点都可以延伸一个举足轻重的行业。

浏览器按下回车键后，发生了太多太多事情，我文字理下图中写不下的重点，详细看下面大图。[大图下载地址](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/浏览器回车后.png)

## DNS

1. 重点是时机。DNS 查询是为了域名和 IP 映射，所以它的时机非常非常靠前，是所有网络活动的第一步。
2. 发挥大作用的是负载均衡。
3. 误解最大的是根 DNS。有镜像 DNS 在，不要拿封锁大陆域名后会怎么怎么样来做文章，技术同学说这话是要丢人的。如果这么做，我能想到的最大影响就是：大陆主动拒绝外界网络，外界都不愿意打开大陆访问，其实都是自由的选择。
4. httpDNS 可以解决 DNS 污染和劫持问题，还有运营商偷懒导致的跨网非最近节点不准确问题，加快域名和 ip 的映射。**一般只用在移动端，技术方案是 hook gethostbyname api 这个环节**。别想着请求前把域名换成 ip，坑很多，尤其是 TLS 证书验证的时候。

## Socket 套接字

1. 网络通信的基石，只要是网络通信，这就是绕不开的大山，这就是中流砥柱。
2. socket 的核心在于 socket 描述符和发送接收缓冲区。发送接收缓冲区不是 socket 特有的，是计算机基础的一部分，我们使用终端的时候，输入指令和参数的时候都在使用这套缓冲区。
3. 大名鼎鼎的**三次握手**就在 socket 建联的时候发生的。**成也萧何败也萧何**，因为 socket 是内嵌计算机的底层服务，也包括 TCP。所以即使三次握手已经拖慢了互联网这么多年，但依旧无法做升级。Quic 的出现就是忍不了 socket tcp 的**队头阻塞**而破釜沉舟的东西。
4. socket tcp 的详细过程，可查阅之前文章：[TCP 数据传输过程分析](https://www.yigegongjiang.com/2020/TCPTranslation/)
4. 谈 socket 离不开端口，谈端口离不开进程。可查阅之前文章：[Shell 和进程-两种进程创建方式](https://www.yigegongjiang.com/2022/shell/#%E4%B8%A4%E7%A7%8D%E8%BF%9B%E7%A8%8B%E5%88%9B%E5%BB%BA%E6%96%B9%E5%BC%8F-By-fork-amp-exec)
5. socket 本身可查阅之前文章：[IM 和 Socket 的关系及 Heart 的必要性](https://www.yigegongjiang.com/2020/im/)

<!-- more -->

## 用户网络

1. 基站的圆形辐射覆盖范围有限，做火车的时候就是在频繁的更换基站。尤其偏远的地方，在火车周边每隔一段距离就有一个基站。基站应该是中国铺的最广的民用基建了，这也是运营商升级缓慢的原因，成本很大。
2. 国内 sim 卡，在国外有的能上 twitter 有的不能上，是虚拟隧道的那一端到底在国内还是国外的原因。
3. 疫情期间能知道哪些人经过了什么地方，就是因为基站的背后就是路由器和互联网，手机到哪就知道人到哪了。我疫情初期听过这么一个案例：一个人黄码了，然后他是内部工作人员，就去删了自己手机号的行程数据，黄码就立刻变绿码了。3 年过去了，现在健康码的操作权限还是很大，不仅可以手动删数据，还可以手动加数据，可以指定哪些人展示什么码。

## 七层五层网络协议

1. 网络通信的基石，和 Socket 一样是中流砥柱。不过 socket 那一层需要复杂的编码，这一层只能躺着使用，可以窥看它的实现，有不爽的地方也无能为力。
2. 会话层的 TLS 是 HTTP 和 HTTPS 的差异，这一块集中了密码学的知识。TLS 的耗时是非常可观的，要想躺着减小耗时，就尽快升级到 TLS1.3。其次，尽快用 ECDHE 代替 RSA，ECDHE 减小了签名大小、解密时间，还支持抢跑。
3. 传输层的 TCP 详细过程，可查阅之前文章：[TCP 数据传输过程分析](https://www.yigegongjiang.com/2020/TCPTranslation/)
4. 这里有两个重要的 case，就是 MTU 和 MSS。传输层的 MSS 会拆包，叫分段，IP 层的 MTU 也会拆包，叫分片。当包在网线传输的时候，其他的路由器属于三层设备，可以将包解到第三层即 IP 层，这个时候也就拿到了 MTU 范围内的全部包数据，如果包数据不符合当前路由器的 MTU 大小设置，可以将包抛弃，或者拆成更小的包后继续传给服务器，服务器会根据编号进行重组(分片重组)。包里面可以设置一个参数，当路由器 MTU 过小需要拆包的时候，不允许拆包，强制直接丢弃。因为 IP 层面的拆包，是风险很大的，这个时候包里面没有了传输层的头部，有可能会服务器侧各种拦截。
5. MAC 层会通过 ARP 拿到下一跳的路由器或者主机的 MAC 地址。MAC 地址并不是电脑本身的一部分，而是网卡的一部分。在电脑主机启动后，网卡驱动会对网卡进行初始化，这个时候网卡有一片 ROM 区域会被驱动配置。后续拿的 mac 地址，其实就是从网卡的这个 rom 区域拿的。
6. UDP 和 TCP 的根本差异，不在于稳定性和使用方式，而在于包大小。如果每个包都很小，那么 UDP 要比 TCP 传输的更高效。只是有些包比较大，UDP 传输的话，一个包丢了就得全部重传。TCP 可以通过稳定的复杂机制来保障快速重传。比如 DNS 查询，就是通过 UDP 来发包的。DNS 查询很重要，为什么没有使用 TCP 的稳定通道？就是所有数据就一个包大小，丢了就丢了，没有回执就是丢了嘛，那就再重传一下。

## NAT 路由

1. 核心啊，能上网就靠它了。大图里面详细说了 NAT 的流程，包括 MAC 层 MAC 地址与 网络层 IP 地址的变化，以及 NAT 映射表和端到端隧道穿越的实现。

## 浏览器渲染

1. 这里重点说了 V8 对 javascript 支持。详细可查阅之前文章：[Shell 和进程-解释编译混合型语言](https://www.yigegongjiang.com/2022/shell/#%E8%A7%A3%E9%87%8A%E7%BC%96%E8%AF%91%E6%B7%B7%E5%90%88%E5%9E%8B)
2. 图片渲染部分，以及如何实现渐进式图片，可查阅之前文章：[文件存储差异 - 编码](https://www.yigegongjiang.com/2023/unicode/)

## CDN 加速

1. 和 DNS 一样，CDN 负载均衡服务器非常有用处。
2. 要提高加速的命中率。即预推要及时，减少回流的概率。
3. 缓存策略要控制好。可以通过 HTTP head 在客户端和服务端同时做缓存。

## 大图来了

![浏览器回车后](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/浏览器回车后.jpg)


