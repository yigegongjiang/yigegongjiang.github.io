---
title: 通过 Swift Package 制作二进制库
date: 2023-12-15 16:18:54
categories:
- 技术
tags:
- Swift
keywords:
---

Swift Package Manager(SPM) 已经被苹果放置于一个很重要的位置，在[历史文章](https://www.yigegongjiang.com/2023/SwiftCommandEnv/#Package)中对其做了一定的分析，Package 目前的定位和 Xcode Project `.xcodeproj` 同级别。SPM 不仅仅用于替代 Cocoapods，而是 Apple 后期语言研发生态的一部分。
鉴于 Xcode 对 Package 的支持，可以很方便的将项目中的代码进行组件化设计，做一定的逻辑分析即可拆分成多个 Package，这在开发过程中非常有利于项目的架构、单测、可持续性。
以前若这样做，需要对 Cocoapods 有深入的了解，这是一个比较复杂且细节的过程。通过 SPM 只需要 `New - Package` 即可，将复杂度从项目级别压缩到文件级别。

SPM 出现后，很多 Pods 模块通过增加 `Package.swift` 配置文件可以很方便交由 SPM 管理。但对于直接通过 `Package.swift` 创建的独立库，并没有方便的方案转为 Pods 管理，还是需要走一遍 Pods 的流程。
不过，这里做一个预言，Pods 终将被 SPM 取代，因为 **SPM 是更具有可持续性和生态深度耦合**，在 Xcode 整合(编辑/搜索/联调等)、多平台兼容，源码管理、二进制库/仓库管理、CI等方面，都具有得天独厚的优势。
相比 Pods 完善的脚本自定义能力(基于 ruby 的生态)，SPM 是有一定的短板，但不推荐。
很多大公司在做统一基建的时候，会深度魔改 Pods。简单的东西越做越复杂的原因，除了增加一些"又不是不能用"的功能，还有就是在复杂度提升后打的各种补丁。
虽然这么说，Pods 完善的自定义能力，SPM 也一样可以做到，毕竟这些能力很大部分都属于扩展能力，如插件。执行流程中大家都可以在 xcodebuild 等相关命令的任意位置，随时可以写一些定制脚本做插入执行。

初期，Swift Package 只能做源码(开源)共享，后面增加了 `binaryTarget` 能力，可以在提供了二进制库(framework/xcframework)的情况下，直接通过 SPM 做分发。(闭源共享)
但二进制库从哪里来，普遍的方案还是通过 `.xcodeproj` 的形式编译导出 framework/xcframework 库。
Swift Package 虽然支持导出动态库和静态库，但流程上还不彻底，并不能直接交付使用，下面对此做一些解释说明。

<!-- more -->

# 0x01 闭源framework说明

在描述 Swift Package 制作二进制库之前，先简要描述下苹果生态里面闭源二进制库的设计。
初期，大家都使用 `.a` 制作静态库，因苹果生态管控严格，除系统库可使用动态库外，开发人员是不允许在上架项目中使用动态库的。
后面因为桌面小组件(Widget)等功能的开放，苹果没有开放多进程能力，而是通过 `.framework` 动态库在主应用和小组件之间进行通信。即没有在主应用中通过子进程开发小组件，而是小组件本身就是独立 app。
同期，`.framework` 也支持封装静态库，即可以不用 `.a` 了，通过 `.framework` 可以同时支持 静态库 和 动态库。
再往后，随着 Swift 的发布（OC-Swift混编）、Widget 功能升级，通过对 `.framework` 进行签名(不允许通过 dlopen 动态调用动态库)并内置到主应用中的方案一直持续到现在。
这中间，Apple 也设计了 xcframework 用于代替 `fat framework`，以方便兼容 Apple 所有产品线的研发设计。
当前还有 `.tbd/.tdlib` 等动态库类型，更多用于系统库或者 `Mac 可执行文件`等平台，需要将库防于特定文件夹位置，这里不多介绍。

整体来说，时至今日，`.framework` 就是苹果生态里二进制分发包的表现形式。
Xcode 里的 `xcworkspace` 是个大管家，可以非常方便的将各个 project/target 进行组织和联动，其核心也是在处理不同 project/target 产出的 `.framework` 包。(Pods 是自动化的接管 xcworkspace)

至此，可以看下 `.framework` 里面都有啥。
```
.
├── Headers
│   ├── ProjLibrary-Swift.h
│   └── ProjLibrary.h
├── Info.plist
├── Modules
│   ├── ProjLibrary.swiftmodule
│   │   ├── Project
│   │   │   └── x86_64-apple-ios-simulator.swiftsourceinfo
│   │   ├── x86_64-apple-ios-simulator.abi.json
│   │   ├── x86_64-apple-ios-simulator.swiftdoc
│   │   └── x86_64-apple-ios-simulator.swiftmodule
│   └── module.modulemap
├── ProjLibrary
└── _CodeSignature
    └── CodeResources
```

* Info.plist: 信息文件，存储适用平台、打包环境等信息
* binaryName(ProjLibrary): 用于和其他项目做链接/动态绑定的库(静态/动态)，是核心产物。
* CodeSignature: Apple 严格规定所有的动态库都需要签名，以防止动态下发动态库后任意执行。
* Headers: 这是 OC 时代的头文件。让其他项目知道当前库有哪些接口，IDE 也会根据这些头文件给予自动化提示。
* Modules: module 化推出很多年了，一直都是不温不火。swift 推出后，对库进行 module 化是必要前提(默认)。编译会更高效，OC 环境下可以使用 `import` 导入 module 化的库(非 include)。
* Modules/xxx.swiftmodule: Swift 的头文件。上面说到 Headers 是 OC 的头文件，这里是 Swift 的。Headers 里面如果有 N 个公开头文件，就需要 N 个 .h 输出。Swift 只需要这里列出来的几个文件即可。IDE 会进行解析。

对于库来说，只要有二进制产物即可，即上面的 binary(ProjLibrary)。但若给其他人使用，还需要单独提供 api 文件。
苹果就将 api 和 binary 合并在一起，还可能封装图片等资源库，一起放在了 framework 里面。用于交付给他人使用。 

# 0x02 Package 输出 Binary 卡点

通过 `new - project - framework`，可以很方便的建立一个 `.xcodeproj framework` 工程并打包。
这里我们说的是通过 Package 打包 framework（`new - package`），毕竟如前面说到的，Apple 很重视 Package 并将其提高到了 `xcodeproj` 同级别。Package 已经自成生态，而不仅仅是和 Pods 做功能对其。

Swift Package 天然支持动静态库，这是一开始推出的时候就和 Pods 做对齐的能力。这在 Xcode 高度集成的生态里非常好用，如下图：

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202312151847236.png" width="30%">

Package 可以放在很多位置，以和主项目配合。Xcode 会自动进行 `.framework` 中 头文件 和 二进制库 的链接工作。

但如果想从 Package 直接输出闭源的 `.framework` 以供其他项目使用，就不太方便了。准确来说，是至今(23年底) Apple 还没有做这一块的支持。
以下是对 Package 直接进行 `Xcode Build`/`swift build <-c release>` 等方式，导出 framework 的结果：
```
Build/Products/Debug-iphonesimulator/
├── MyLibrary.o
├── MyLibrary.swiftmodule
│   ├── Project
│   │   └── x86_64-apple-ios-simulator.swiftsourceinfo
│   ├── x86_64-apple-ios-simulator.abi.json
│   ├── x86_64-apple-ios-simulator.swiftdoc
│   └── x86_64-apple-ios-simulator.swiftmodule
└── PackageFrameworks
    └── MyLibrary.framework
        ├── Info.plist
        ├── MyLibrary
        └── _CodeSignature
            └── CodeResources
```

需要的核心二进制文件所处的路径为：PackageFrameworks/MyLibrary.framework/MyLibrary，但是头文件并不在 framework 里面，而是和 PackageFrameworks 平级。
这样就使得 framework 并不能直接交付，因为其他项目没有头文件的 api，无法使用。

以上是 Package 的产物配置为 动态库 的结果。如果是 静态库，则更加严重：
```
Build/Products/Debug-iphonesimulator/
├── MyLibrary.o
├── MyLibrary.swiftmodule
│   ├── Project
│   │   └── x86_64-apple-ios-simulator.swiftsourceinfo
│   ├── x86_64-apple-ios-simulator.abi.json
│   ├── x86_64-apple-ios-simulator.swiftdoc
│   └── x86_64-apple-ios-simulator.swiftmodule
└── PackageFrameworks  <-empty
```

并没有输出 framework，只剩下已经编译好的 .o 目标文件。

其实，若 Package 按上图中直接集成项目中，输出的产物也是这样，只是 Xcode 默认会进行目标文件的链接或者头文件查询等工作。
只是当独立进行 Package framework 输出的时候，上面两个问题官方是没有给予支持。

# 0x03 Package 输出 Binary 方案

对于 Package 输出 动态库 framework，目前的问题是头文件不在 framework 里面，无法直接给其他项目使用。
前面我们已经对 framework 里的内容做了分析，Headers 是 OC 的头文件产物，Package 是明确不支持 OC 的，只能写 Swift 代码。所以不用关心。
我们只需要把 swiftmodule 头文件夹，按照 framework 的标准复制到 PackageFrameworks/MyLibrary.framework/Modules 里面即可。Modules 里面的 `module.modulemap` 在这里也不需要了，它主要是为 OC module化服务的，如 sub module 等。
所以，如果是 xcode 打包，我们可以在 scheme 的 `Build Post-Actions` 或者 project 的 `Build Phases Run Script` 中加入一些脚本，把 swiftmodule 复制到 `.../xx.framework/Modules/` 里面，则 xx.framework 即可以作为动态 framework 给其他项目使用。

对于 Package 输出 静态库 framework，目前的问题是 framework 都没有生成，只有编译好的 .o 目标文件。
静态 framework 是对 .a 和 头文件的封装，一个可行的办法是通过 `ar -crs libXXX.a XXX1.o XXX2.o` 命令，将所有的 .o 进行压缩 .a 文件（需要移除 .a 后缀，framework 主动识别，不要 .a 后缀）。
然后把 .a 文件和 swiftmodule 放到新建的 xx.framework 中。
虽然可行，但也看得出来，这有些复杂。还有 sign/info.plist 文件没有处理，当然实际上它们也并不需要。
这个方案有些复杂，但是一个曲线救国的方案却很方便，就是 `xcframework`。

# 0x04 xcframework

简单介绍一下 xcframework，它是苹果生态里面最近几年提出来的`多平台二进制`输出方案。framework 是`多架构二进制`输出方案。

通过 xcode 或者 xcodebuild 打包的 framework，虽然看起来是一个文件，但其实是 `fat framework`，它里面可能封装了多个架构(arm64/x86_64)的二进制，以供不同架构平台(真机/模拟器)使用。
通过 file 命令可以看见其支持的架构：
```
file ...//Build/Products/Debug-iphonesimulator/ProjLibrary.framework/ProjLibrary

// (Mach-O universal binary with 2 architectures 表示支持两个架构)
...//Build/Products/Debug-iphonesimulator/ProjLibrary.framework/ProjLibrary: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit dynamically linked shared library x86_64] [arm64:Mach-O 64-bit dynamically linked shared library arm64]
// (x86_64)
...//Build/Products/Debug-iphonesimulator/ProjLibrary.framework/ProjLibrary (for architecture x86_64):  Mach-O 64-bit dynamically linked shared library x86_64
// (arm64)
...//Build/Products/Debug-iphonesimulator/ProjLibrary.framework/ProjLibrary (for architecture arm64):  Mach-O 64-bit dynamically linked shared library arm64
```

如果提供给其他项目的 framework 只有 x86_64 架构，真机运行的时候，编译就会报错。同样，只有 arm64 架构的话，非 M 系列的 Mac 电脑的模拟器也编译不通过(M1等系列的模拟器已经支持arm64)。
所以在以往，大家交付 framework 的时候，都需要手动或者通过 CI 的方式执行脚本，把 模拟器/真机 的独立 framework 进行合并，输出 fat framework 后做交付。
这些脚本是如何实现的，这里就不解释了，可以查询更多相关介绍。

需要注意的一点是，多架构还是多平台，和动态或者静态没有关系。framework 本身是 动态库 或者 静态库的封装，但它们都有 多架构/多平台 方案。

后面苹果的生态原来越多样和复杂，就推出了 xcframework 方案，即不在按照架构进行 framework 的整合，而按照 平台 来。这一套方案已经实施很多年了，这些年大家提供的包一般都是按照 xcframework 来，不用过多质疑。
具体实现也非常简单，就是 `m.xcframework` 是一个文件夹，里面的每个子文件都是对应平台的命名如 ios/iosimulator/mac 等，子文件夹里面是具体的单架构 framework 包。下面是一个示例：
```
MyLibrary.xcframework
├── Info.plist
├── ios-arm64  <- iOS真机使用的
│   └── MyLibrary.framework
│       ├── Info.plist
│       ├── Modules
│       │   └── MyLibrary.swiftmodule
│       │       ├── Project
│       │       │   └── arm64-apple-ios.swiftsourceinfo
│       │       ├── arm64-apple-ios.abi.json
│       │       ├── arm64-apple-ios.swiftdoc
│       │       └── arm64-apple-ios.swiftmodule
│       └── MyLibrary
└── ios-arm64_x86_64-simulator  <- iOS模拟器使用的
    └── MyLibrary.framework
        ├── Info.plist
        ├── Modules
        │   └── MyLibrary.swiftmodule
        │       ├── Project
        │       │   └── x86_64-apple-ios-simulator.swiftsourceinfo
        │       ├── x86_64-apple-ios-simulator.abi.json
        │       ├── x86_64-apple-ios-simulator.swiftdoc
        │       └── x86_64-apple-ios-simulator.swiftmodule
        ├── MyLibrary
        └── _CodeSignature
            └── CodeResources
```
实际上，xcframework 并不管 `x.framework` 是不是真的拥有当前平台的架构，它是按照文件夹进行过滤的。如果编译的时候不符合，就报错这样子。甚至每一个具体的二进制文件依旧是包含多架构的 `fat framework`，xcframework 也不会过问。这些信息，也同样会在聚合的 Info.plist 里有标记。
所以这里有个注意点，即最好准确的输出每个二进制文件的架构并输入到对应的子文件夹中，若包含多架构，xcframework 的尺寸会凭空大很多。(一般是通过命令行操作，不会有误差。若通过 xcode 手动打包，得注意 目标设备不要选中 `any ios devices simulator` 等，这会打出多架构的包出来)

对于如何输出 xcframework，也提供了友好的命令，即 `xcodebuild -create-xcframework  -framework pathA/N.framework -framework pathB/N.framework -framework pathC/N.framework -allow-internal-distribution -output N.xcframework`，这是把多个 framework 合并成一个 xcframework 的方式，具体可查阅更多资料。

在上一节说到曲线救国的 Package 输出 静态库 framework，就是巧妙的使用 `xcodebuild -create-xcframework` 命令。具体如下：
`xcodebuild -create-xcframework  -library pathA/N.a -library pathB/N.a -library pathC/N.a -allow-internal-distribution -output N.xcframework`
即待合并的目标文件使用 -library 标记，这样可以将多个 .a 文件聚合为 xcframework，而不用关心手动将 .a 转为 .framework 的复杂问题。（当然，这里还需要手动/脚本将 swiftmodule 复制到对应 framework 的 modules 文件夹中。）

# 0x05 小结

至此，说明了如何在 Package 环境下，快速输出二进制给到其他项目使用。

对于 `fat framework` 或者 `xcframework` 会不会增加线上包大小，不用担心。因为当前平台&架构的包，准备提交的时候以及 App Store(iOS/Mac) 均会进行过滤。准备提交的时候，若参杂 模拟器 包，会不允通过（一般 Pods/SPM 已经默认做了去除。若手动管理，可增加 build phases 脚本进行剔除）。同时包含 armV7 和 arm64，在提交审核后也会被苹果后台根据不同平台进行剔除，App Store 上看到的已经是去除冗余架构后的大小。
但是对于非 App Store 的包，就不一定了。iPhone 有巨魔等自签平台，量很少不用关心。在 Mac 平台上，自签的 dmg 为了适配 Intel 和 M 架构，会同时包含多个架构，这会增加包大小。当然也可以为不同平台提供不同的包，增加少许复杂度。

实际上闭源的 framework/xcframework 不仅仅用于输出二进制给到其他项目使用。
对于大型项目，在长年累积过程中，可能会有好几百个模块库。使用源码的话，会严重增加编译时长(编译一次耗时 1H 不夸张)。
这时候将每个模块直接输出二进制库，对编译时长有质的飞跃。

___

现在不方便眨眼睛，请不要问太多。
