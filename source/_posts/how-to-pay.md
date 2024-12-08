---
title: How to pay
date: 2024-12-09 01:18:03
categories:
- 技术
tags:
- Life
- Work
---

# All pays

[https://stripe.com/zh-sg/payments/payment-methods](https://stripe.com/zh-sg/payments/payment-methods)

Stripe 支持了很多支付方式，可以窥见一些。Stripe 主要对接线上支付，对于很多线下支付的渠道如【nanaco】等电子货币，就看不到了。

## japan

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090122548.png" width="60%">

<!-- more -->

# 货币

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090139321.png" width="60%">

# 电子货币 - 支付链路

Suica、PASMO、nanaco、WAON、Edy（Rakuten Edy）、QUICPay、iD

如上文【货币】-【电子货币】中所描述，E-money（电子货币）是法定货币的 1:n 等值替换，即电子货币 = 法定货币。

电子货币无法凭空产生，需要基于预付费模式工作的，这意味着用户需要先充值然后才能消费。

用户将一定金额的资金存入电子钱包或智能卡中，然后在各种接受 E-money 支付的地点使用这些资金进行消费。

消费方式：实体卡刷卡、Apple/Google Pay、App 出示识别码等等。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090124537.png" width="60%">

# 钱包 - 支付链路

市场上有各种 X Pay，均为钱包。除了 Apple/Google Pay 这种专门为 信用卡、预付卡、借记卡、电子货币 提供聚合服务的产品外，其他钱包均有实际公司主体。

比如常见的 Paypay、wechat、alipay 等等，它们提供各式各样的终端消费能力。

但用户在钱包中的钱并不能凭空产生。

一种方式是依靠【信用卡】，信用卡 的链路最为复杂，下一章会单独讲诉。

一种方式是依靠【预充值】，用户将法定货币预先充值到平台侧。形式有：预付卡、借记卡、平台账户余额。

通过 预充值 的形式，钱包相当于中间平台，为用户的【纸币、硬币】这些法定货币，提供了中间放置平台，以进一步通过该平台向外部商家进行支付。

> 当用户把钱充值到钱包后，钱包就需要一系列的管理功能。一来确保用户账户数额不能有差错，二来确保用户可以随意把钱花出去。 所以平台需要做一个大管家，提供各种安全保障能力。 甚至与，哪个商户若需要对平台用户进行收款，哪个商户就也要在该平台进行商家注册并绑定收款银行账号。 这样，商户的账户也被平台管控，平台就很容易进行交易清算。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090125351.png" width="60%">

# 信用卡 - 预付卡 - 借记卡

## 概要

信用卡关键的能力有两点：

1.  提供个人信用背书，个人没有钱，也能够通过信用卡消费。
2.  有稳定的支付通道，为世界各地的信用卡提供清算能力。

信用卡普及且应用广泛后，整个体系就非常成熟，商家已经对信用卡支付形成依赖。此时，一些不符合 1 条件的用户就没有信用卡，但自身也有钱/能力进行支付，就衍生出了预付卡、借记卡。

预付卡、借记卡 借用 2 支付通道，不提供信用背书，直接用现金支付，从而完成交易。

> 预付卡、借记卡 也包括电子货币。只是在信用卡体系里，预付卡、借记卡 一定依赖 visa、mastercard、… 等支付网络，一定属于 visa、mastercard、…卡。
> 若商户对 个人信用 十分在意，那么只能使用 信用卡 完成交易，预付卡、借记卡 无法支付。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090126692.png" width="60%">

## 品牌贴牌

目前大范围使用的信用卡支付网络，只有 Visa、MasterCard、American Express(AmEx)、Discover、UnionPay、JCB。其中，大部分不发行卡，只提供【支付网络】。支付网络将是下一个章节重点说明的内容。

> American Express 和 Discover 提供发卡服务。即提供支付网络，也下场作为发卡行。<这两位，拥有银行执照>

大众最常见的信用卡，是从银行处申请获取。发卡行一定是具有【银行执照】的金融机构，银行天然具有该优势。基本上，所有的信用卡，都是银行发行的。

但还有一种常见的信用卡，是从【钱包】公司处获取，如 Paypay、MerPay 等。这些公司并不具备银行执照，即没有发卡能力。

这是另一种常见的【品牌贴牌】信用卡，即 公司 对接发卡行。公司对用户进行信用评估等审核，并提供品牌福利，而发卡行承担用户的消费风险及盈利。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090127218.png" width="60%">

## 信用卡支付网络

【支付网络】是信用卡交易的核心，也是连接全球所有信用卡的枢纽。

所有的发卡行、收单行、有能力对接支付网络 的机构、单位、环节，其【资质、技术、安全】等都需要满足【支付网络】提供商的要求。

【支付网络】决定了整条交易链路的【规则、安全保障、清算、结算】等。

所有的信用卡，不分地区，通过【支付网络】都可以进行联通、交易。发卡行承担不同币种的汇率、消费额度等工作。

> 发卡行会在交易前进行验卡，防止卡滥用。如有些卡不允许国际支付，会拒付。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090128277.png" width="60%">

## 著名的国际支付服务提供商

上文中，很多商业产品都会通过【支付服务提供商】来对接【支付网络】最终完成交易。以下是一些有名的提供商列表：

via [https://developer.apple.com/tap-to-pay/regions/](https://developer.apple.com/tap-to-pay/regions/)

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090128854.png" width="60%">

## Apple Pay 如何支持 EMV & FaliCa

EMV 是 Europay MasterCard Visa 的缩写，是信用卡的协议标准，不同支付网络的信用卡均遵守该协议。

所以，信用卡在世界各地支持 EMV 读卡器的机子上均能【插卡支付】并消费。

但是在 Touch(contactless) 方面，日本提前广泛使用了 FaliCa 标准，没能很好的支持 EMV，导致前期不管国内还是国外的信用卡，都不支持【非触控支付】。

在技术专区里，会重点说明 FaliCa 的这段历史，以及 Quicpay/iD 如何解决这个问题。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090129489.png" width="60%">

## 信用卡的不同支付链路

信用卡最终需要【发卡行】通过【支付网络】转账到【收单行】。

发卡行 依靠哪些信息来判定一张信用卡的合法性，是十分关键的。

在不同的 信用卡 使用场景中，发卡行 从【支付网络】侧获取到的卡信息是不一样的。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090129298.png" width="60%">

# 特别支付场景介绍

## Tap to pay

> 基于 NFC 技术，可以【技术专区】中详细了解实现过程。

通过**移动设备终端**(iPhone/Android) 识别 visa 等实体卡或者 apple pay 等**物理硬件**，**获取到卡片的【代理卡号/虚拟码】，并完成支付**的方式。

1.  通过 NFC 协议，完成数据读取
2.  移动设备会有一个 app，用来对接 nfc api，完成数据读取、服务器交互等工作
3.  Android 对于 NFC 较开放，可以方便实施。
4.  iPhone 使用了 Core NFC 中【卡读取】的能力。对个别国家和供应商有开放。

### example

via [https://www.apple.com/business/tap-to-pay-on-iphone/](https://www.apple.com/business/tap-to-pay-on-iphone/)

### **apple 供应地区说明**

via [https://developer.apple.com/tap-to-pay/](https://developer.apple.com/tap-to-pay/)

### **Stripe 支持地区说明**

via [https://docs.stripe.com/terminal/payments/setup-reader/tap-to-pay?platform=ios](https://docs.stripe.com/terminal/payments/setup-reader/tap-to-pay?platform=ios)

## Alipay 碰一下

> 基于 NFC 技术，可以【技术专区】中详细了解实现过程。

1.  支付宝给商家提供 NFC 卡模拟芯片。用户侧的支付宝 app 充当读卡器，读取商家的 NFC 芯片信息。app 获取商家信息后，进行网络处理，完成支付。
2.  以往的支付，读卡器处于商家侧，如 Tap to pay、POS 机等。只要商家具有稳定的网络，就可以完成支付。
3.  alipay 碰一碰方案，读卡器处于用户侧 app 中。这就需要用户侧具有稳定的网络，以完成支付。

# 技术专区

## Visa - 支付网关和清算平台 - 跨国际交易

1.  Visa 不发行卡，不对接个人用户。它只对接银行、大企业主。
2.  Visa 负责两个银行之间的资金流动，它会使用一天的固定汇率(高于实时汇率)加上自身的手续费进行计价。
3.  因为 Visa 自身做的比较大、有信任、打广告，所以只要牵涉到跨国之间的终端用户级别的资金流动，都会有 Visa 的影子。
4.  如果仅仅是本地企业、同一货币，那么不需要使用 Visa，就可以省去手续费。如银联。
5.  所有的 Visa 信用卡交易，都需要过 Visa 的网关进行清算。最终也不是实时扣款，visa 会在结算时间到来后，统一将所有银行的账单进行清算。

> 扩展：

1.  Visa 和 SWIFT 是两条赛道。SWIFT 主要处理两个国家之间银行界资金的流动。是直接流动，金额更大、级别更高。SWFIT 处理的金钱要比 Visa 多的多。
2.  SWIFT 一日处理 5W亿美元的交易，Visa 一年处理 10W亿美元的交易。交易量不在一个量级。

Visa 等一众【支付网络】的具体运转，详见上文中的【信用卡支付网络】章节。

### 非银行如何对接【支付网络】

Stripe 对 Visa 的一些解释：[https://stripe.com/zh-sg/resources/more/what-is-visa#shui-zai-shi-yong-visa](https://stripe.com/zh-sg/resources/more/what-is-visa#shui-zai-shi-yong-visa)

Stripe 在 2015 年开始直接对接 Visa，不在走【收单行】对接：[https://www.businesswire.com/news/home/20150729005841/zh-CN/](https://www.businesswire.com/news/home/20150729005841/zh-CN/)

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090134169.png" width="60%">

## Apple Pay

### 原理概要

Apple 提供的【apple pay】方案，是将用户的信用卡信息，使用“令牌化”方案，通过【代理卡号】的方式存储于 iPhone 设备，后续将 DAN 提供给读取者，读取者使用 DAN 完成后续的支付链路。

> 代理卡号（Device Account Number, DAN） 不存储卡号、有效期、cvs，是银行侧生成的一个 code。银行会根据这个 code 在结算的时候进行卡账户校验。
> 
> 每次进行 apple wallet 录入的时候，都会生成一个新的 DAN。

使用 DAN 的业务方：

1.  Apple Wallet：iPhone 作为 NFC 卡模拟，供系统级别的 Wallet app 使用。
    1.  wallet app
        1.  将银行卡信息存储到 iPhone 中（录入 DAN）
        2.  有外部 NFC 读卡器的时候，wallet 被激活，完成用户身份认证，并读取 DAN 并提供给外部 NFC 读卡器。
        3.  外部读卡器获取到 DAN 后，对接 Stripe 或者卡服务商。最终在发卡行完成 DAN 的校验，完成扣款和资金转移。
2.  Native app：通过 passkit sdk，swift/oc 对接 sdk 在 app 内部完成【支付信息】的读取
    1.  native app 绑定 【Merchant ID】出口，未绑定的 id 不允许支付。
    2.  native app 弹出 apple pay 弹窗，获取 pay token（公钥加密）
    3.  将 token 给到 self server 或者 stripe 等平台，这些服务平台需要对 token 进行私钥解密。
        1.  私钥的来源有一套比较复杂的流程
    4.  server 将解密后的信息给到【发卡行】，发卡行完成校验及扣款事宜。
3.  Web app：
    1.  整体和 native app 一致
    2.  因为不在 app 内部，缺少必要的 Merchant ID 信息。而是在 safari 中唤醒 apple pay 支付，所有多了一个【Identity】认证
        1.  通过公私钥证书来完成
        2.  该认证，主要用于对服务器进行确认，进而获取商户信息。后续流程和 native app 一致。

### 技术流程

### native - self server

1.  Apple Developer 申请 Merchant ID（id）
2.  dev 申请 CRS 文件（本机绑定私钥 private key），在 developer 后台绑定 id 生成 cer 证书。
    1.  dev 将 cer 证书安装到本机后，导出 p12 文件（包含公私钥 public + private key）
    2.  将 p12 给到 self server
3.  native app 在 xcode 中绑定 id
    1.  该行为使得 app 可以确定允许的商户，可绑定多个 id
    2.  web app 没有这一步，所以需要 【identity】认证
4.  native app 完成 apple pay 弹窗并获取 payment token
5.  token 给到 server，server 通过 p12 拿到 private key，对 token 进行解密后，提交发卡行进行验证扣款。

### native - stripe

1.  Apple Developer 申请 Merchant ID（id）
2.  在 stripe 后台申请 CRS 文件后，在 developer 绑定 id 生成 cer 证书，再将 cer 证书上传到 stripe 后台。
    1.  stripe 这里就拥有了 p12 （private key + public key）
3.  native app 调用 stripe api，完成 apple pay 弹窗 等所有操作，直接获取支付结果
    1.  stripe sdk 将 payment token 给到 stripe server 完成 token 解密
    2.  stripe server 进行发卡行提交
    3.  stripe 处理支付结果并返回给业务

### web - self server

和 native 基本流程一致，在 id 绑定证书环节，除了 【Apple Pay Payment Processing Certificate】cer 证书外，还需要以下：

1.  Apple Pay Merchant Identity Certificate
    1.  商户需要通过该证书向 apple 获取 Merchant ID 的详细信息
    ```
    // example

const https = require('https');
const fs = require('fs');

function getApplePaySession(validationURL, callback) {
    const options = {
        url: validationURL,
        method: 'POST',
        cert: fs.readFileSync('path/to/merchant_identity_certificate.pem'),
        key: fs.readFileSync('path/to/merchant_identity_private_key.pem'),
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            merchantIdentifier: 'merchant.com.yourdomain.yourmerchantname',
            domainName: 'yourdomain.com',
            displayName: '您的商户名称'
        })
    };

    const req = https.request(options, (res) => {
        let data = '';
        res.on('data', (chunk) => { data += chunk; });
        res.on('end', () => { callback(null, JSON.parse(data)); });
    });

    req.on('error', (e) => { callback(e); });
    req.end();
}
    ```
    
2.  Merchant Domains
    1.  需要将必要的 apple 文件放置于 domain 服务器上，供 apple 对 domain 进行验证。未验证通过的 domain 无法进行 apple pay 支付。
    
## NFC

NFC（近场通信）的技术原理基于无线电频率识别（RFID）技术，使用磁场感应来实现在设备间的通信。NFC 设备在13.56 MHz频率上操作，通常用于非接触式数据传输，距离范围非常短，通常在几厘米内，传输速率慢，在 400kbps (50kb/s)左右。

### 技术原理

识卡器发出信号（电磁感应），激活了终端（手机自动点亮），然后进行数据交互，并可能需要机主进行身份验证，最后完成信息的交互（支付、上公交车、开门等）。

NFC 有三种工作模式：点对点通信模式、读卡器模式、卡模拟模式。又分为【主动模式】和【被动模式】，其中一个设备提供射频场，另一个设备利用这个射频场进行通信。

使用 NFC，需要两个终端，一个做控制器，用于发射磁场来识别信息。一个做无电源的数据芯片，通过接收到的磁场来感应并传输信息。

对于 NFC 设备来说：

1.  点对点通信：两个 NFC 设备相互交换信息。
2.  读卡器模式：一个 NFC 作为控制器，读取其他 NFC 芯片中的信息。
3.  卡模拟模式：一个 NFC 作为数据芯片，其他读卡器可以读取其中的信息。

现代电子产品中，Android 和 iPhone 都支持 NFC 技术，手机作为 NFC 设备，使用读卡器模式和卡模拟模式，已经可以完成很多事情。

1.  当作为 NFC 控制器的时候，手机可以主动的读取外部 NFC 芯片中的信息，也可以将必要的信息写入到外部 NFC 芯片。（物流中，可以通过手机对商品挂载的 NFC 芯片进行记录）
2.  当作为 NFC 芯片的时候，手机可以模拟一个 NFC 芯片，通过软件将信息提前写入手机中，其他读卡器就可以直接读取手机中的信息。（可以实现门禁卡等）

Android 对 NFC 的 API 开放较多，app 可通过 api 来控制 NFC 进行 读取、写入、模拟 的操作，来实现快捷的智能家居、门禁卡等场景。

iPhone 上则比较保守，在【卡模拟】、【卡读取】方面，都有不少限制。

### 门禁卡

普通门禁卡：

1.  门禁卡中有 【微芯片】（存储卡片的识别信息和其他数据）和 【天线】（用于接收和发送无线信号）。
2.  门禁卡靠近门禁系统的读卡器 → 读卡器会发出一个射频信号 → 信号通过天线供电给门禁卡上的微芯片 → 微芯片被激活,通过天线将存储在芯片上的识别信息发送回读卡器 → 读卡器接收到信息后，将数据传输给后端的门禁控制系统。

NFC 门禁卡：

原理基本和普通门禁卡一样，不过，NFC 提供了更高的安全性。它支持双向通信，卡和识卡器之间可以通信。它们之间会进行密钥交换，通过对称、非对称加密来完成数据的安全传输。相比普通门禁卡，NFC 门禁卡会更加的安全。

> NFC 是一种普适性的技术方案，手机也可以作为 NFC 终端。这里就可以把 NFC 门禁卡的信息保存在 手机中，使得手机可以充当 NFC 门禁卡的功能。

蓝牙门锁：

有些 app 会通过 蓝牙的方式，和门锁连接。这在智能门锁中非常常见。因为距离很远，就可以连接上。而 NFC 需要非常短的距离(4cm)才能通信。

### 移动支付

通过 NFC 进行移动支付，主要有三种方案：

1.  卡模拟方案。mobile 录入支付卡信息，被外部读取器识别
    1.  系统级别的支持，如 apple pay。开发人员没有掌控能力。用户只能通过 apple wallet 录入银行卡信息，然后通过 apple pay 进行支付。
    2.  应用级别的支持。Android 支持的较好，iPhone 限制很多。
        1.  iPhone 在 iOS 18.1 放开了该限制，app 可以将支付卡信息写入 app 中，支付的时候调用 app 完成支付。
        2.  不对普通开发者开放，需要和 apple 签订商务协议，支付费用，一般都是支付中间商如 Stripe。并且只对个别国家开放。
2.  读卡器方案。mobile 作为读取器识别外部实体卡（信用卡等）。Android 支持的叫好，iPhone 限制很多。
    1.  Apple Tap to pay。商家可以在自己的手机中，打开 m app，然后用户把信用卡、iWatch 靠近手机，即可完成支付。
        1.  普通开发人员没有太多的控制能力。也需要签订商务协议，一般都是支付中间商如 Stripe。它们提供 SDK 并和 Apple Api 交互完成支付。
        2.  Apple 和 Stripe 中间商会对地区等有限制。只在少有的地区开放了 Tap To Pay 能力。
    2.  alipay 碰一碰。非常聪明的通过 NFC 实现支付的方案。
        1.  支付宝给商家提供 NFC 卡模拟芯片。用户侧的支付宝 app 充当读卡器，读取商家的 NFC 芯片信息。app 获取商家信息后，进行网络处理，完成支付。
        2.  以往的支付，读卡器处于商家侧，如 Tap to pay、POS 机等。只要商家具有稳定的网络，就可以完成支付。
        3.  alipay 碰一碰方案，读卡器处于用户侧 app 中。这就需要用户侧具有稳定的网络，以完成支付。

### Apple iPhone NFC

简单介绍一下 iPhone 对 NFC 支持的历史：

*   WWDC 2017：引入 Core NFC，并具备 NDEF 标签【**读取**】 功能。
*   WWDC 2018：在较新设备上对 NDEF 消息进行后台标签读取。
*   WWDC 2019：重大扩展，允许 NDEF【**写入**】，支持 ISO 7816、ISO 15693 和 MIFARE 标签，并支持自定义命令。
*   WWDC 2020：多标签检测，VAS 协议支持和 ISO 15693 标签的后台读取。
*   WWDC 2021-2023：专注于稳定性、性能提升和小幅增强，没有重大 API 更改。
*   WWDC 2024：支持【**卡模拟**】。

### EMV vs FeliCa

EMV：基于 NFC Type A/B 设计的通信标准。NFC 的【读卡器】和【卡芯片】均使用 EMV 协议。

FeliCa：Sony 开发，基于 NFC Type F 设计的通信标准，速度快，成本高，仅在日本大范围使用。NFC 的【读卡器】和【卡芯片】均使用 FeliCa 协议。

这两者，是使用 NFC 实现的两套不兼容的通信协议标准。详细技术解释可参考【Quicpay】章节。

## Quicpay - iD 为什么称为【电子货币】

Quicpay 官网: [[https://www.quicpay.jp](https://www.quicpay.jp)](https://www.quicpay.jp/)

日本很早期，地铁等交通就很繁华，对于人流量大的场景，过关卡排队就需要很长时间。

One day，Sony 基于 NFC Type F 研发了 FeliCa 协议的高频无线通信技术。

FeliCa 主要用于 Touch/Contactless【非触控】场景，Pasmo、Suica 都是基于 FeliCa 实现的刷卡，特点是：速度快。

很快，FeliCa 成为日本在【非触控】刷卡领域的事实标准，所有能刷卡的地方，都是基于 FeliCa 实现。

前面介绍 NFC 的时候，说到 NFC 需要两个终端（读卡器、卡模拟）才能工作，日本所有的读卡器也都是基于 FeliCa 实现的。

而卡模拟一侧，包括 Visa 信用卡、Pasmo 公交卡、电子货币实体卡 等，都是支持 FeliCa 的。

前面介绍 Apple Wallet 对接 Quicpay 的时候，流程图中有提到 Quicpay DAN。在 Apple Pay 还没有进入日本之前，Quicpay、iD 就已经发行实体卡进行消费，同时 Visa 等信用卡也都是支持 Quicpay/iD 支付。

交易的链路和现在相比没有改变，依旧是读卡器通过 FeliCa 读取卡信息后提交到 Quicpay 后台，后台通过 收单行 对接 Visa 支付网络，完成交易。

只是这个时候没有 DAN，DAN 安全码是 Apple Wallet 特有的产物。

但是 FeliCa 的硬件成本比普通的 NFC Type A/B 协议高，国际上普遍使用的都是 EMV 协议。EMV 是基于 NFC Type A/B 实现的通信协议。

在刷卡过程中，数据是需要加密的，而这套加密规则，也是和 FeliCa/EMV 的设计绑定的。

此时，日本基于 FeliCa 的读卡器，在【非触控】刷卡的时候，就无法读取基于 EMV 协议设计的国际信用卡。

> 但是对于插卡消费是没有影响的。 Visa、masterCard 等官方平台，制定的通信规则就是【信用卡基于 EMV 协议】。所以日本的信用卡本身绝对是符合 EMV 协议的。 所以日本的信用卡在插卡消费的时候，也同样使用 EMV 协议。因为 FeliCa 协议只在【非触控】场景下使用。 So，这个时期，外国游客在日消费，可以使用信用卡插卡消费，但无法使用 Touch/Contactless【非触控】刷卡消费。

> 对于在日本申请的信用卡实体卡，本身就是支持 EMV 和 FeliCa 两种通信协议。在日本的时候，读卡器可以插卡识别 EMV，也可以【非触控】识别 FeliCa。 当日本人出国在境外刷卡，外国的读卡器都是支持 EMV 插卡 &【非触控】识别，所以日本信用卡在国外使用完全不受影响。

对于国外信用卡在日本不能很好使用【非触控】的体验问题，并没有好的解决办法。FeliCa 历史已久，在速度方面很优秀，Pasmo 等众多卡都使用这个协议，根子是不能替换的。

所以日本开始升级读卡器，截至目前，很多读卡器也都同时支持 EMV 和 FeliCa 两种协议的 NFC 识别了。

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090137924.png" width="60%">

同时，随着 Apple/Google Pay 的发展，现在 Apple Wallet 也支持 Quicpay/iD 支付了，在日本称为【电子货币】。

在 Apple Wallet 中见到 Quicpay/iD 的场景，都是信用卡场景。即 wallet 中，一张卡的右下角，同时具有 Quicpay + Visa 或者 iD + masterCard Logo。

有些钱包公司发行的卡，仅仅支持 Quicpay 或者 iD，就没有 Visa 或者 masterCard Logo 了，即这个卡就不能在国外使用啦。

> 但从 Apple Wallet 中 Visa、masterCard 支持 Quicpay/iD，被称为【电子货币】或许有些奇怪。 若从 Quicpay/iD 自身的功能场景出发，它们本身就是提供预充值的实体卡而后刷卡消费，的确属于【电子货币】。 只是 信用卡 虽然走了 Quicpay/iD 的支付通道，又的确和【电子货币】没关联，其实依旧属于【信用卡支付】。 so，Quicpay/iD 被称为【电子支付】，完全是根据其自身的原始功能，下的定义。

最终在说明一点，在日本银行发行的信用卡，基本上都是和 Quicpay 或者 iD 合作的。有些钱包公司借壳发卡行发行的卡，有可能没有 Quicpay/iD。

这里有一份列表，via [https://atadistance.net/apple-pay-japan-credit-cards/](https://atadistance.net/apple-pay-japan-credit-cards/)

<img src="https://raw.githubusercontent.com/yigegongjiang/image_space/main/blog_img/202412090138969.png" width="60%">

___
