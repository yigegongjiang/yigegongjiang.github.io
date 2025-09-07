---
title: iPhone 侧载安装
date: 2025-09-07 22:35:07
categories:
  - 工具
tags:
  - Life
  - iOS
---

iPhone 上安装一个非 App Store 的 app 太难了，Apple 生态基于【公私钥】非对称检测，使得专业的行内人员也无法很好的在开发机上随意安装三方 app。
这里有一些对普通人来说也比较方便的实施方案。安全性上，除了被安装的 ipa 包本身可能有风险，其他链路很安全。

> 1. 非 App Store 版本的 app，来源有 github、三方分享平台等。
> 2. app 安装需要 ipa 包。App Store 上下载的就是 ipa 包，这里需要通过 github、telegram 等渠道获取。

# 普通、小白、非专业、非 IT 用户

通过 [AltStore](https://altstore.io/)，就可以非常方便的安装 ipa 包了，具体流程需要查看下文档。
需要 Mac / Window 电脑 + 个人的 AppleId。安全性上，非常安全，技术方案是：

<!-- more -->

1. Apple 开发人员需要 Apple Account 才能开发/发布 app 到 App Store。前些年 Apple 开放了门槛，使用个人 Apple Id（free account） 也能够进入开发流程，但是 app 不能上架 App Store。
2. free account 有限制：一个 iPhone 同一时间最多只能安装 3 个 app + 一周只能激活 10 个 app（每个 app 都有唯一 id，一周最多 10 个 id）
3. 通过对三方渠道获取的 ipa 使用 free account 进行重签名（骗过 iPhone 公私钥检测），假装自己是 app 的开发人员，就可以安装到自己的 iPhone 上。

缺点：

1. 每个 app 安装 7 天后就打不开了，需要通过 AltStore 将 iPhone 连接上 Mac/Window 后重新安装下（free account 限制）。
2. 同一时间最多安装 2 个 app（free account 合计 3 个，有一个是 AltStore app）。

## 更进一步

上面列出的 2 个缺点，一般人用用也够了。如果想方便一些，通过 [SideStore](https://sidestore.io/) 可以更进一步的解决一个小痛点(其他缺点均无法解决)：
SideStore 的重签流程放到了 iPhone 上，可以脱离 AltStore Mac/Window 了。
这样，可以很方便的在 iPhone 上完成 7 天失效的【续重签名】，不用打开电脑了。

# Apple 开发人员

Apple 开发者都有 Apple Account，AltSotre 除了支持个人 AppleId，也支持开发者账号。上面的两个缺点就都解决了。
开发者证书签名的 app 有 1 年有效期，足够用了。

## 单点登录 Apple Account

有些开发者账号是公司提供的，管理员关闭了账号密码直接登录，就无法在 AltStore 使用了。这个时候要进行深度探索了。

有一个 app/ipa，叫【全能签】。主动设置开发证书后，全能签通过证书来完成 ipa 重签名并安装到 iPhone。原理也是和 AltStore 一样，都是 ipa 重签名，骗过 iPhone 公私钥检测。

这个流程的具体操作是：

1. 通过 AltStore 安装 全能签（全能签 a，7 天有效期）
2. 设置 全能签（开发证书等）
3. 通过 全能签 安装 全能签（全能签 b，1 年有效期）

第三步通过全能签 a 自举 安装的 全能签 b，有 1 年有效期。
这个时候，就可以把 AltStore、全能签 a 等全部卸载了，仅保留 全能签 b。
后续所有的 ipa 包，都可以通过 全能签 b 来安装，有 1 年有效期，够用了。

# 其他

之前越狱很有名，现在基本灭绝了。不过衍生出来一个新的分支，叫【巨魔】，又称【半越狱】。
巨魔 可以方便的安装 ipa 包，因为走的系统漏洞，app 不会过期，长久有效。
巨魔 技术上需要使用系统漏洞，而 Apple 会更新补丁，所以最新系统基本都无法使用 巨魔。
而巨魔之所以存在，因为非 App Store app 依旧有一批市场。

So，如果想了解上面介绍的【侧载】安装的应用场景，还是需要先进入这片字面意义上，不正规/灰色的领域。

---
