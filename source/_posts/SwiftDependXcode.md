---
title: 在 Xcode 中更好的使用 Swift
date: 2023-12-16T02:48:42+08:00
categories:
- 技术
tags:
- Swift
---

在 OC 时代，通过 .h 头文件以及 include 标记，还有那长长的 api 名称，可以很方便的意会和找到某个类。使用 Xcode 十年，在 Vim 的加持下，基本没用过啥快捷键。
最近在 Swift 里面一顿拾掇，理解源码也太复杂了。
1 个 class 在 3 个文件中增加了 5 个扩展并实现了 4 个协议，1 个文件中包含 400 行注释 500 行代码 10 多个杂七杂八的 enum/struct/class/extension/protocol，去他妈的。

一直都是夸奖 Swift 的，作为后现代语言，集百家长补百家短。
但不好的编程习惯对 Swift 来说会是灾难，文件名不在作为当前文件内容的强约束、无需 import 后的 extension 可以写在任意位置、enum/struct/class 是值类型值语义引用类型引用语义搞不愣清后的混用，等等，这些都会骤然增加 Swift 源码的理解。

不得已对 Xcode 进行一些调教。
目标是快速**查阅继承、搜索、实现协议方、三方库文档**等代码阅读操作。

<!-- more -->

# 0x01 搜索快捷治理

vscode 的搜索是出奇的快和方便查阅。很喜欢做的一件事情，是把公司几百个仓库 down 下来，通过 vscode 搜索要抄的作业。这时候通过搜索简短的注释级别的语意，就能找到期望的代码片段，AI 都得落泪。

对于 xcode，感觉就像老人，多操作几个功能就菊花圈转啊转的。搜索符号、文本的时候也经常有卡顿。可能是电脑太差了吧。
最近把搜索切到快捷键后，感觉快了很多。这应该是不用鼠标操作 UI，避免了主线程的卡顿。xcode 中主线程易卡死，最烂 IDE 不是吹的。

新增的快捷搜索如下：
```
⌥C  ->  Find Call Hierarchy
⌥A  ->  Find Ancestor Types
⌥C  ->  Find Conforming Types
⌥D  ->  Find Descendent Types
⌥S  ->  Find Selected Symbol in Workspace
⌥T  ->  Find Selected Text in Workspace
```
在 setting - Key Bindings - filter【find】进行修改，使用 Option 做前缀，这样不会产生快捷键冲突。后缀使用语意单词的首字母，方便记忆。
在需要搜索的单词上按下快捷键，就可以切到 搜索 区域展示搜索结果了（**无需全选单词，xcode 默认会做单词内容全选**）。

对其中的几个搜索做下介绍：
Find Ancestor Types: 展示当前 class 的逐个父类(祖先类)。其中，协议也属于祖先类，会一并展示。
Find Descendent Types: 和上一个相反，展示子类。不过无法展示协议的实现类。和上一个有差异。
Find Conforming Types: 展示当前协议的实现类。其中若 A 协议继承 B 协议，搜索 B 也会将 A 展示出来。
Find Selected Symbol in Workspace: 根据符号进行搜索，相比文本搜索会更准确。如 函数、属性 的搜索，直接展示所有使用方。

这几个快捷键，基本覆盖找代码过程中的大部分场景。里面还有一些小技巧，如**前后缀及单词匹配、大小写敏感、展示方法和属性**等。

## 缺陷
对于 Procotol，若 ProtocolB 继承 ProtocolA。无法通过以上 `Find` 操作从 ProtocolB 找到其祖先协议 ProtocolA。这是 Xcode 的缺陷。
这个有一个补救的办法，是通过下面说到的 DocC 来查阅。DocC 文档的末尾，一般有一个 `Conformed to` 列表，用于标记当前 Procotol 实现了哪些祖先 Protocol。

## 技巧 1 Xcode Index

对于 `⌥T` 文本搜索，任何条件下使用都没有问题。但是其他的搜索如符号、祖先等，就需要 xcode 先解析项目生成 Index 索引，然后才能通过读取 db 缓存来使用。
这里的解析项目，是不需要编译通过的。只需要 Xcode 把项目的词法分析操作完即可(默认打开项目就会解析)。

但这里也会有一个小问题，就是多个 Group 下有大量无法编译通过的文件，Index 索引生成会失败。
这时候可以通过新建 Target，将文件添加到不同的 Target 中。可以解决 Index 索引失效问题。

## 技巧 2 Scheme 去除干扰

如 `Xcode Index` 中描述，对于有些项目，如 github 源码、自己建立的测试工程等，某些原因下可能无法编译通过。
这个时候项目里面会有很多 error，使用搜索的时候，会有很多刺眼的红色警告条干扰。

这个时候可以操作 `Manager Schemes - show`，将所有 scheme 取消勾选。因为 xcode 已经把项目解析完成，搜索依旧是可以使用的。这样可以有效的去除警告干扰。

## 技巧 3 Search Scopes

巧妙设置 Scopes，可以大范围缩小部分文本的搜索范围。搜索范围可以设置为部分主项目文件夹和任意三方库的关联。
有个小技巧，是按着 Control 选中需要搜索的文件夹或者三方库，然后右键，会有一个快捷添加 Scopes 的入口。

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202312160601060.png" width="30%">

## 技巧 4 正则搜索

配合 ChatGPT 写**正则搜索**，也非常棒。如果需要对正则结果使用文本替换功能(replace)，最好在 vscode 里面操作，效果会更好。

# 0x02 Swift 官方源码

上面说到的搜索，仅仅对于有源码的当前项目或者三方库进行使用。Swift Foundation 等模块，只能通过其生成的 api 接口来预览。
这个时候弊端非常明显，就是很难查找当前一个系统类的父子及实现协议之间的关系(如：想要查看 Array 实现了哪些 Protocol 以及父子类)。

因为 Swift 是开源的，这里一个办法是直接获取 Swift 源码。具体方法可以参考这篇文章。[swift-5.9-RELEASE源码编译（Xcode）](https://juejin.cn/post/7291555600854171683#heading-15)

整套流程非常耗费时间和磁盘大小(62.5G)，若非查看 cpp 实现源码，可通过 lite 版本进行 swift 源码查阅 [swift5.9.2_rawcode_lite](https://github.com/yigegongjiang/swift5.9.2_rawcode_lite)。
当然，cpp 源码包含非常多的有价值内容。如 LLVM 项目中有整套编译器和平台架构的实现。Swift 项目中有对 ELF/Mach-0 Image 等详细操作流程的实现。

# 0x03 使用 Assistant 视图

在编辑窗口使用默认快捷键 `⌃⌥⌘↩` 可打开 **Assistant** 视图。在做 Storyboard/SwiftUI 等 UI 面板开发的时候经常用里面的 auto 功能以打开对应源码文件。
其中还有一些如 Procotol/extension/superclass 等，毕竟方便展示当前打开页面的清单。

尤其里面有一个 `interface` 能力，可以展示当前 swift 源码对应的头文件。这样可以边看源码边看接口定义，非常方便。

# 0x04 首选 DocC 文档

苹果开放了 DocC 能力，可以快速输出任意项目/三方库的接口文档。具有和官方文档一样的体感，非常好用。

DocC 文档可通过 `⇧⌘0` 快捷键快速体验。通过 Product - Build Documentation 可快速输出项目接口文档。

# 0x05 Bookmark

新版 xcode(15) 增加了书签 Bookmark 功能。可以在文本任意行右键找到入口，在搜索面板的左侧找到书签总入口。

以前，都是通过打断点然后把该断点设置取消状态的方案，在断点总入口里面刻意做到标记。把断点当书签用。
现在有了单独的书签，展示上也更加友好。这些 IDE 必备的功能，虽迟但到吧。

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202312160617272.png" width="30%">

# 0x06 加餐：vim

xcode 自建了 vim 的支持。vim 对中文场景一直有诟病，切换输入法的瞬间秒变灾难。vscode/终端 均是如此。
但是 xcode 完美支持中文。在普通状态中文场景下，按键会自动表意为英文并执行对应的命令，不会变灾难。
xcode 是我见过的对 vim 支持最友好的 IDE。

如今，我经常中文写作，对于 md 文档也希望通过 xcode 来管理。我的方案是这样的：
1. 建立一个命名为 EditBlog 的 workspace 和 library，这样可以提供一个最小化的 xcode 项目模版。
2. 将 md 文档的文件夹设置 alias，并将 alias 放置于 EditBlog 项目中。在 Xcode 里面操作 `add files to EditBlog` （不选中 target，不选中 copy，使用 folder reference）。这样就完全将 md 文档接入到 xcode 工程里面了。 
3. 如果误操作了 xcode 的什么项目快捷键，当前工程也不会有什么错误响应。最主要的是什么时候想写字了，就即可。

优点：
1. 实时同步。xcode 里面可以实时创建新 md。若外部创建了新 md，xcode 里面也会同步展示出来。（不选中 target，不选中 copy，使用 folder reference）
2. 打开项目方便。打开 xcode，通过 recent 打开 EditBlog 项目即可。
3. 快捷键误操作，不会触发什么 error。因当前 xcode 项目是最小化的模版，且 md 没有链接到 target 中。

___

自然增长即无刻意操作情况下，自然能达到的数量级。

一个千万级日活的 app，新增一个重要的功能入口。
那么如何评估这个功能的数据是否优秀，不能看 UV 或者 DAU。即使什么都不做，这个量级的用户基数也会有上百万的点击。

这对国家层面数据指标的鉴别也有重要参考，因为这里的基数更大。正向数据表达的可能是增长，也可能是降低。
