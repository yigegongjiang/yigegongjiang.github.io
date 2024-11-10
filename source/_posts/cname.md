---
title: DNS 的 CNAME 是如何工作的
date: 2024-11-10 22:54:14
categories:
- 技术
tags:
- 网络
---

今天配置 cloudflare 中站点的 dns，遇到一些关于 cname、tls、cloudflare 代理相关的问题，做了下梳理。重点有下：
1. cname 负责提供目标 ip，在机制上类似【权威域名服务器】
2. 目标 ip 所在的服务如果非自行掌控（比如强行设置一个目标域名），很可能无法工作。
3. cloudflare 设置 cname 的时候有一个【代理】功能。这个设计非常糟糕，完全脱离了 cname 的本意。

以下介绍中，使用 cloudflare 配置 `test.yigegongjiang.com` 的 cname 为 `httpbin.org`，默认不配置【代理】。

<!-- more -->

# CNAME 是做什么用的
DNS 解析中，通过 `dig A test.yigegongjiang.com`，`dig A httpbin.org` 会有如下返回（参数 A 表示返回目标域名的 ipv4）：

```
   // dig A test.yigegongjiang.com
 
   ;; ANSWER SECTION:
-> test.yigegongjiang.com.  300  IN  CNAME  httpbin.org.
   httpbin.org.    60  IN  A  54.243.34.18
   httpbin.org.    60  IN  A  34.206.181.91
   httpbin.org.    60  IN  A  3.222.34.231
   httpbin.org.    60  IN  A  35.172.59.156
   httpbin.org.    60  IN  A  54.237.204.19
   httpbin.org.    60  IN  A  184.73.239.81
 
   // dig A httpbin.org
   ;; ANSWER SECTION:
   httpbin.org.    60  IN  A  3.222.34.231
   httpbin.org.    60  IN  A  34.206.181.91
   httpbin.org.    60  IN  A  54.243.34.18
   httpbin.org.    60  IN  A  184.73.239.81
   httpbin.org.    60  IN  A  35.172.59.156
   httpbin.org.    60  IN  A  54.237.204.19
```

可以发现，如果需要找到 `test.yigegongjiang.com` 的 A 记录，一定需要先找到 CNAME 记录，再通过 CNAME 指向的域名继续寻找目标 ip。

所以这里提出 **CNAME** 的第一个作用，就是【设定 IP】。使用 github pages 搭建博客的时候：
1. 购买了域名 m ，希望将 m 域名映射到 n.github.io Blog 域名上。
2. 后续直接访问 m，ip 被重定向到 n.github.io，从而完成 Blog 的访问。

其次，**CNAME** 还有一个作用是【负载均衡】。如使用 xx 云平台部署服务：
1. 在 xx 云平台上的多个边缘节点部署了服务，并统一使用域名 m 向用户提供服务。
2. 无需自行搭建【权威域名服务器】，通过 xx 云平台提供的 n.xx.com 做域名 m 的 cname 指向。
3. 用户请求域名 m 的时候，DNS 解析会进入 n.xx.com，xx 云平台负责根据用户的地理位置通过 n.xx.com 提供动态的 ip，实现【边缘访问】和【负载均衡】。

Anyway，不管 CNAME 通过中继域名实现【设定 IP】，还是做【负载均衡】，CNAME 本质上都是为初始域名提供 ip。
不像 A/AAAA 记录，强制设定了一个固定 ip。CNAME 提供了一个【缓冲层】，可以做更多的事情。

这里也会出现一些问题，通过缓冲层获取到的 ip 所在的服务器，可能无法很好的处理我们的 m 域名请求。

# CNAME 目标服务器注意事项

假设我们回到了没有 tls/ssl 安全验证的 http 场景，将 `test.yigegongjiang.com` 的 cname 指向 `www.google.com` 或者 `www.facebook.com`，可以访问 Google 和 facebook 吗？
理论上可以，实际并不行。如果把 cname 指向 `httpbin.org`，却又可以正常访问。
进行 http 请求的时候，主机域名 `test.yigegongjiang.com` 会通过 【host】字段携带到目标服务器，此时目标服务器完全可以主动拒绝服务，因为不是它们自己的域名。
而`httpbin.org`能正常访问，是它没有限制请求中的 host。

回到 https 场景，有了 [tls/ssl 安全证书](https://www.yigegongjiang.com/2023/signature/#%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%EF%BC%9ASSL)，又变得不一样了。
同样进行上面的设定，不管 cname 指向哪个域名，浏览器都会弹窗告诉我们不安全。因为 `www.google.com` 返回的 tls 证书，验证的域名是 `*.google.com`，而用户访问的域名是 `test.yigeogngjiang.com`，不匹配！证书验证不通过！
甚至于有些目标域名，连安全弹窗都不会弹起。
> 服务器如何返回证书，是通过 tls 认证中的【SNI】标记识别具体服务的（一个服务器可以开 n 个服务，有 n 个证书）。
> 如何服务器都不认识 `test.yigegongjiang.com`这个 SNI 标记，那么它可能连 tls 证书都不会返回。

上面提到的 google、facebook、httpbin 服务，虽然不认识 `test.yigegongjiang.com` 这个域名，但还是返回了他们各自默认的 tls 证书。此时，如果用户选择强制信任证书：
1. google、facebook 回到了 http 时候一样，因为 host 不匹配，主动拒绝服务了。
2. httpbin 也回到了 http 时候一样，可以正常访问。因为 httpbin 默认没有限制 host。

对于一台 Nginx 配置的目标服务器（假设默认域名是 a.com），如果需要完美兼容 `test.yigegongjiang.com` 这个域名的 cname 映射，需要做两件事：
1. 识别到 SNI 是 `test.yigegongjiang.com` 后，需要返回 `yigegongjinag.com` 的 tls 证书，供浏览器进行安全链验证。
  a. 或者返回一个 tls 证书，该证书同时包含 `a.com` 和 `yigegongjiang.com` 的认证。
2. 将 `a.com` 和 `test.yigegongjiang.com` 这两个 host 同时指向同一个内部的 port 服务。
  a. 或者不识别 host，所有 host 都指向同一个内部的 port 服务。

上面的两个方案，如果是特殊的服务场景倒还能实现，如 github page 提供了 static blog 服务，可以根据我们配置的域名来进行设置，从而完成访问。
对于 `fly.io`、`koyeb.com` 这些云平台，自定义域名就是它们的收费项，那肯定不会放开这个口子。

## 通过 代理 实现 koyeb.com 云平台自定义域名？

如果有一个免费的中间代理，我们访问 `test.yigegongjiang.com` 的所有请求，这个中间代理帮我们将所有流量正向代理到 koyeb.com 平台上，不就可以既不买 koyeb 的服务，又能实现自定义域名了吗？
cloudflare 提供了这个能力，也是它的安全、缓存等一系列功能的起点。
cloudflare 配置 dns cname 的时候，有一个【代理】选项。选中后，当前配置虽然看起来是 cname，但完全脱离了 cname 本意。
1. 此时不是 cname 了，通过 dig 工具会发现此时返回的是 cloudflare 的 ip
2. 访问域名的流量，会被 cloudflare 转发到 cname 指向的域名上，这里的 cname 充当【代理】的角色。

实际上，依旧访问不通。显然 koyeb 这些云平台不会留下这么大的空子给别人钻。
因为 cloudflare、aws 这些正规的代理服务，在请求目标服务的时候，都会提供完善的信息已告知目标服务：我是代理。
它不仅把原始的域名等信息提供给了 koyeb，还会把自己的信息都提供给 koyeb。所以，koyeb 直接拒绝服务就行了。

当然，我们可以自己搭建一个【猥琐】的【三级代理】，将 cloudflare 那边的代理流量打过来，然后再去请求 koyeb，此时【猥琐代理】完全装作正常用户的浏览器，那 koyeb 的确是察觉不出来的。
此时，cloudflare 又能够捕获到完整的数据，实现缓存、边缘节点、cdn 等等能力。

___


``````
