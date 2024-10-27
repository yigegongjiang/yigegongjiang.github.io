---
title: 移动端换端方案与场景还原
date: 2024-10-27 15:38:17
categories:
- 技术
tags:
- iOS
- Swift
- Work
---

# 换端能力

在移动应用生态中,换端(App Switching)是一个常见的需求。同一台设备上，有以下三个端: `M(Web网页)`、 `A(移动应用 app)`、 `B(待唤端的应用 app)`。 当 M 或 A 需要跳转至 B 时,有两种技术方案: URL Scheme 和 Universal Link。

## URL Scheme
1. App 端先行验证能力:
   - A 可以通过`canOpen`方法预先判断 B 是否已安装,且此操作不会触发跳转
<!-- more -->
2. Web 端无先行验证能力:
   - M(Web) 无法预先判断 B 是否安装
   - 通常采用延迟判断方案:执行跳转后等待2秒,根据页面状态判断跳转是否成功

URL Scheme 存在一个严重的安全问题:不同的应用可以注册相同的 scheme。这意味着当 A 尝试跳转 B 时,可能会被恶意应用 X/Y/Z 截获。正是由于这个安全隐患,URL Scheme 正逐渐被 Universal Link 替代。

## Universal Link
1. 预判限制:
   - M 和 A 都无法预先判断 B 是否安装
2. 跳转控制:
   - A 可以通过跳转参数强制使用 Universal Link 进行跳转
   - 跳转执行后可以判断目标应用是否安装

### 原理 & 挑战
- 应用安装时,操作系统会向配置的 domain 请求资源文件(开发人员提前配置),该文件包含应用约定的 Router 信息
- 如果 HTTPS 请求失败,应用将暂时无法被 Universal Link 唤醒
- Apple 未明确指定重试机制,仅表示系统会在适当时机重新请求

# 微信的二次回跳验证机制

## 二次回跳说明
当应用 A 首次使用微信 SDK 的 Universal Link 功能进行分享时,会出现双重跳转:
```
A -> 微信 -> A -> 微信
```
这种情况仅在首次分享时出现,后续分享操作将直接完成。

## 双重验证有效性
Universal Link 在安全的前提下，成功率较高，能保证其他 App 跳转到我的 App 的成功率。只要我的 App 配置了 Universal Link，大概率其他 App 跳转到我的 App 是没有问题的。
但是，我的 App 也需要跳转回其他 App，此时我的 App 并不信任其他 App（或者说不信任其他 App 的 Universal Link 配置）。
举个分享流程的例子，如果 A 通过 Universal Link 跳转到微信后，微信处理完分享流程。用户点击返回 A，但无法返回，此时就会造成严重的用户体验缺失（Bug）。

## 二次回跳处理
微信采用类似 TCP 三次握手的验证机制，提前打通 A 和微信之间的换端通路。这可能涉及到降级处理，即 Universal Link 降级为 URL Scheme。仍以分享流程为例：

1. A（通过微信 SDK API）使用 Universal Link 跳转到微信。
    - SDK API 增加了选项控制，指定使用 Universal Link 方式。如果失败，直接降级到 URL Scheme 进行跳转，此时不会有二次回跳验证。
2. 微信拉取服务端资源，将 A 的 URL Scheme 和 Universal Link 拉取到本地（A 提前在微信的开发者平台进行了配置）。
3. 微信通过 Universal Link 跳转回 A。
    - 指定使用 Universal Link。如果失败，说明 A 的 Universal Link 配置有问题，降级到 URL Scheme 通知 A。后续 A 和微信的操作（分享操作仍需继续）都通过 URL Scheme 完成。
4. A（通过微信 SDK API）判断微信通过 Universal Link 成功唤起了自己，表明 A 和微信之间的 Universal Link 通路畅通。
    - 后续所有的换端操作，全部通过 Universal Link 完成。
5. A（通过微信 SDK API）发起真正的分享换端，微信被唤醒后执行分享流程（朋友、朋友圈、收藏等），然后回跳到 A。
6. ...
7. 之后，所有换端操作都直接通过 Universal Link 进行，不再需要之前的二次回跳验证。

# Web 向原生 App 发起登录验证

web 页面在需要登陆的场景，可能需要跳转到 app 中完成登陆，然后回到到网页中。

## 处理流程
1. 唤起阶段:
   - Web 页面 (url_a) 通过 Universal Link 或 URL Scheme 唤起App
   - 携带必要的身份标识参数

2. 登录处理:
   - 方案一: 服务端传递Token
      - App 将 token 和身份标识发送至服务端
      - App 通过 `openURL(url_a)` 在 Safari 中刷新网页（不会新开页面）
      - Web 根据本地存储的标识参数，从服务器获取 Token。

   - 方案二: 换端传递Token
      - App 将 Token 拼接到 `url_b_{path}` 中，通过 openURL 唤醒。
      - Safari 会新开页面，新的页面是登录成功后的落地页。

# Delay Deep Link（场景还原-一般用于广告投放）

通过 Universal Link，如果目标 App 未安装，此时可以跳转到 Web 中间页并转跳到 App Store。希望用户下载 App 并首次打开后，立即跳转到当初 Universal Link 的承接页。这也叫【场景还原】。
核心原理是，全新的 App 在第一次被打开后，需要通过某种途径，找到之前 Universal Link 中标记的落地页信息。

1. 方案一：本地
    - 在使用 Universal Link 进行换端时，发起换端的一方负责将换端链接存储到用户的剪贴板中。
    - 当目标 App 打开后，自行读取剪贴板并进行落地页跳转。剪贴板有预探功能，可以先不触发用户的权限弹框，提前知道是否有需要的口令内容。

2. 方案二：设备标识（Device ID）
    - 这里的 Device ID 并不是真正的设备 ID，而是大家共同遵守的一套规则，可近乎唯一地标记一台设备，并保证漂移率很低。可以根据设备的开机时间、升级时间、系统版本等信息，组合成密钥字符。
    - 在使用 Universal Link 进行换端时，发起换端的一方负责将 Device ID 和落地页信息存储到云端。
    - 当目标 App 打开后，自行获取 Device ID，然后从云端回拉落地页信息。

一般来说，这些场景都牵涉到资金投放，需要尽可能高的识别精度。所以方案都是混合使用，有一个匹配上就算【场景还原】成功了。

___
