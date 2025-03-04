---
title: passkey
date: 2024-12-16 13:49:33
categories:
- 技术
tags:
- 网络
- 计算机基础
- Work
---

在很久以前的 [数字签名](https://www.yigegongjiang.com/2023/signature/#%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%EF%BC%9APassKey) 文章中，有介绍过 passkey。当时主要是为了说明【公私钥】。目前很多服务商都支持 passkey 作为【登录】和【多重认证】的手段，于是再重点介绍一下 passkey。

# FIDO

FIDO 是 passkey 技术流程的执行标准，明确了使用公私钥认证的方式来实现用户安全登陆。其中，私钥的存储、获取也进行了安全规定的制定。

> FIDO（快速身份在线）是指由 FIDO 联盟开发的一套开放标准，用于增强在线身份验证的安全性和便捷性。FIDO 标准主要通过公钥密码学替代传统的密码，以减少欺诈风险并提高用户体验。FIDO 2 是该联盟的最新标准，支持 WebAuthn 和 CTAP，允许用户通过生物识别、安全设备或 PIN 进行无密码认证。

# 安全终端有哪些

FIDO 约定了私钥的存储和获取。其中，存储必须是对用户不可见、不可导出，即对人类而言，是完全隐秘的黑盒。常见的 passkey 私钥源头是：

1.  手机，如 iPhone、Android
2.  电脑，如 Macos、Windows
3.  passkey security key device，专用的移动安全设备，具有 USB 等插口，可以插入电脑中。
<!-- more -->

# 哪些不是 passkey

对于用户的安全登陆，web 站点会提供很多【多重认证】的方案，如

1.  SIM 短信验证码
2.  email 验证码
3.  App 短期有效的验证码
4.  通过 App 对 web 二维码扫码验证
5.  通过外部安全软件如 mac/iPhone 的 password app 生成 30s 有效的 Verification Code。

以上这些，有的场景可以直接用于用户注册、登陆，有些场景可以作为用户的【二次安全验证】，但它们都不属于 passkey。

passkey 的首要条件有：

1.  基于人类的安全识别，如指纹、面容、眼球
2.  基于公私钥，不能无密钥，也不能是对称密钥。

> 如 password app 生成的 Verification Code，是在设置的时候，server 生成对称密钥并把密钥办法给 app。后期 server 和 app 基于同一时间按照同一个密钥生成 Verification Code，并根据该 Code 是否相等来做验证。技术手段属于【对称加密】。
> 示例（参数包含：对称密钥、算法、长度、过期时间）：`otpauth://totp/PeerAuth:M?issuer=PeerAuth&secret=6T7PVVPTZRPWLXXTLZPACRM52QIUAQPE&algorithm=SHA1&digits=6&period=30`

# passkey 可以做什么

诞生的使命就是做用户认证。即可以用于【二次认证】，也可以用于【一次认证】。即：

1.  可以通过 passkey 直接登陆 server 服务。用户不需要拥有账号密码，只要有 passkey 就可以了。
2.  用户已经输入账号密码后，通过 passkey 进行二次安全确认。
    * 这一步，可以减少互联网诞生以来，各种类型的验证码对用户的影响，甚至上面提到的那些常见的【多重认证】对用户登陆的阻断性干扰。

# 技术流程

## 当前设备作为 passkey 终端，如 Mac

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412161356392.png" width="60%">

用户注册 passkey 或者 绑定 passkey（用户开始阶段有无 web 站点的账号，皆可）：

1.  web 申请 passkey 并提供【本机认证】入口，如 “创建通行密钥” 入口按钮。点击后进入 Macos 系统认证页面。
2.  用户确认后，操作系统将在系统安全元件中存储【私钥】，并把【公钥】给到 web。
3.  web 将公钥上传到 server

后续，用户登陆或者二次验证的时候：

1.  web 提供【通信密钥登陆】入口。用户点击后，web 向 server 申请【挑战(一个复杂的计算)】。
    * server 使用公钥对【挑战】进行加密，并返回 web。
2.  web 唤醒并进入 Macos 认证页面（【挑战】也一并给予），用户进行指纹验证通过后，操作系统用指定的私钥对【挑战】进行解密，返回 web 最终是否解密成功。
3.  web 根据操作系统返回的结果，来判断私钥是否合法。如合法，则顺利进入 web 站点（登陆成功，或者二次验证成功）。

## 移动设备作为 passkey 终端，如 iPhone

整体流程和 上面 一致，只是多了一个终端：iPhone。在上面创建 passkey 的时候，还有一个【换部设备】的入口，点击后调用系统 api 并展示系统 passkey FIDO 弹窗：

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412161357253.png" width="60%">

> // 二维码解密后内容
> 
> FIDO:/246202778134775422047073255772941326219821733578350172887457888589614449494332690387759208522109390157096037518904470510056446099681926406642559495045890109321447142660

iPhone 扫码后，会打开 passkey 移动端系统弹窗。iPhone 在设备上生成公私钥，私钥在安全元件中存储，并把公钥通过【一些通信方式】给到 web 站点。

1.  初期，使用 websocket，即 chrome 等浏览器打开 websocket，iPhone 通过 websocket 把公钥给到 chrome。chrome 进而给到 web 站点。
2.  现在，已经不用 websocket 来。iPhone 会通过私有协议，把公钥给到 macos，macos 通过 系统 api 返回给 chrome/web 站点。

除了以上多了一个二维码，供 iPhone 用来识别之外，其他就没有流程上的变化来。

核心差异点就是：【私钥到底存在哪】。当然，存在哪里，哪里就需要作为后续提供私钥的源头。即：

1.  用户若使用 iPhone 绑定 passkey。
2.  后续，用户也需要使用 iPhone 主动扫码，来进行公私钥验证。

当然，这一块也有很大的优化空间，比如 Apple 可以实现操作系统级别的【私钥同步】，把 iPhone 中的私钥同步到 Macos 中，这样 mac 也可以直接进行公私钥验证，就不需要 iPhone 扫码了。【目前是否有这个能力，还不清楚。】

# 用户负担

如果完全使用 passkey 作为服务商的用户注册（拒绝 账号、密码），可能遇到的问题如下：

1.  在自己的 apple 全家桶中，passkey 可以无缝同步。但如果哪一天迁移到了 android、window 后，因为没有账号密码、又没有私钥，就无法登陆服务了。
2.  在自己的设备体系中使用良好。如果哪一天需要在同事的电脑上登陆，就没有 passkey 私钥了。

最近一些年，可能更常见的方案，还是在已有的账号密码的用户体系下，新增【绑定 passkey】能力。

若以后有更丝滑的【私钥迁移】能力，那么伴随互联网发展长河的账号密码体系，可能会成为历史。

___


