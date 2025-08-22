---
title: Xcode Symbolic Debug
date: 2025-08-22 18:57:21
categories:
  - 技术
tags:
  - Swift
  - iOS
---

Xcode 的 Symbolic Breakpoint（符号断点）在排查问题的时候非常好用，尤其在三方闭源库联调的时候。

> 在闭源三方库中，如果能根据公开的 Api 找到一些有用的信息，还是非常 nice 的。

不知道什么时候开始，符号断点的内容格式非常严格，不然无法被断点。虽然 Xcode 给了如下提示，但那么多符号，很难短时间处理好：

```
Xcode won't pause at this breakpoint because it has not been resolved.
Resolving it requires that:
• The symbolic name is spelled correctly.
• The symbol actually exists in its library.
• The library for the breakpoint is loaded.
```

<!-- more -->

简单来说，之前通过快捷键可以将当前指针所在的 Symbolic 快速录入到搜索框中进行搜索（xcode 支持符号检索），然后把输入框内容复制到 Breakpoint 就能进行符号断点了。现在死活断不到。
需要：绝对准确的符号签名，包括 static、参数、返回值 等等全量信息，少一点就断不成。

快速的方案，是在 Debug 中执行 `image lookup -rn xxx` 来查找整个工程中特定符号的内容，在其中找到准确的符号签名后，再设置到 Breakpoint 中。这里使用的 xcode lldb 命令，输出内容一不小心就闪瞎眼。

还有一个比较快捷的方案是使用 [https://github.com/DerekSelander/LLDB](https://github.com/DerekSelander/LLDB)，通过其 `lookup xxx` 命令，就可以非常整洁的整理出来所需符号的完整签名信息。copy 一下就能使用了。

```
// e.g.

(lldb) lookup initApp
****************************************************
1 hits in: EleSDK
****************************************************
static EleSDK.Ele.initApp(key: Swift.String, apiURLString: Swift.Optional<Swift.String>) -> ()
****************************************************
4 hits in: ManagedConfiguration
****************************************************
+[MCLazyInitializationUtilities initAppleIDSSOAuthentication]
__61+[MCLazyInitializationUtilities initAppleIDSSOAuthentication]_block_invoke
__61+[MCLazyInitializationUtilities initAppleIDSSOAuthentication]_block_invoke_2
objc_msgSend$initAppleIDSSOAuthentication

```

最后，也比较一下 `image lookup -rn xxx` 的结果：

```
(lldb) image lookup -rn initApp
1 match found in /xxx/Ele-ios-demo-swift-gfeqjptpbrkqcrborvnyugpagjfl/Build/Products/Debug-iphoneos/Ele-ios-demo-swift.app/Frameworks/EleSDK.framework/EleSDK:
        Address: EleSDK[0x000000041efc] (EleSDK.__TEXT.__text + 237308)
        Summary: EleSDK`static EleSDK.Ele.initApp(key: Swift.String, apiURLString: Swift.Optional<Swift.String>) -> ()
4 matches found in /Users/hailv/Library/Developer/Xcode/iOS DeviceSupport/iPhone17,3 18.6.2 (22G100)/Symbols/System/Library/PrivateFrameworks/ManagedConfiguration.framework/ManagedConfiguration:
        Address: ManagedConfiguration[0x00000001a1bc9a2c] (ManagedConfiguration.__TEXT.__text + 225740)
        Summary: ManagedConfiguration`+[MCLazyInitializationUtilities initAppleIDSSOAuthentication]
        Address: ManagedConfiguration[0x00000001a1bc9ab4] (ManagedConfiguration.__TEXT.__text + 225876)
        Summary: ManagedConfiguration`__61+[MCLazyInitializationUtilities initAppleIDSSOAuthentication]_block_invoke
        Address: ManagedConfiguration[0x00000001a1bc9b18] (ManagedConfiguration.__TEXT.__text + 225976)
        Summary: ManagedConfiguration`__61+[MCLazyInitializationUtilities initAppleIDSSOAuthentication]_block_invoke_2
        Address: ManagedConfiguration[0x00000001a1cc8a20] (ManagedConfiguration.__TEXT.__objc_stubs + 22656)
        Summary: ManagedConfiguration`objc_msgSend$initAppleIDSSOAuthentication

```

找到 func 签名，挺费事儿的。

---
