---
title: Swift 脚本开发环境搭建
date: 2023-12-13 11:27
categories:
- 技术
tags:
- Swift
- Swift三方源码系列
keywords: Swift,Package,Package.swift,SPM,Swift Package Manager,命令行开发,脚本开发,ArgumentParser,进度条输出,异步等待,系统命令,Process
---

不推荐使用 Swift 写脚本，和 Python 比起来，该生态链相对匮乏，开发耗时会增加很多。
但对于偏 Swift 的同学来说，这也的确是更可控和方便维护的方式之一。尤其对于公司内部工具，有问题可以更快的找到原因并处理，不至于手忙脚乱。

相对于 Python 环境丰富的基建资源，使用 Swift 做脚本开发的优点在于**生态的一致性**。
苹果官方抹平了 脚本开发 和 应用开发 的界限，即 开发环境、三方库导入和使用(苹果自身的开源库/三方库)、系统平台 api(存储、网络、协程、并发等，UI api 除外)等，均表现一致。
可以像应用开发的流程一样，进行脚本命令行的开发。

<!-- more -->

# 0x01 Swift 生态

一言两语，先概要描述下 Swift 目前的生态，以明确脚本开发所处的位置。

Swift 做为苹果大力推出的 iOS/Mac 多平台应用开发语言，目前广泛用于开发 app(ipa/dmg)。做为古老的 Objective-C 替代者，基本上对接了 OC 里面所有的 Api，可以直接调用。这对新语言的铺广有很大好处，使用成本有非常大的降低。
但 Swift 也抛弃了 OC 的消息传递机制，使得 patch 等能力很难实现。这也是大公司不愿意迁移的原因，在苹果不允许热更新之后，它们都耗费大精力维护一套不公开的 patch 能力，以在紧急时刻做补救。

Swift 做为后浪，完全集百家之长，也躲过了这些年高级语言已知的不完美，这是相当优秀的点。
至目前(23年底)，已经对范型、协程、并发等高级场景做了完备的支持。
尤其值得关注的是发源地 app 开发，Swift 提供了 SwiftUI 做为原始 UIkit/AppKit 的替代，这是目前非常跟随潮流的 UI 基础建设。

Swift 以跨平台为前提，也同样可以做服务端开发、输出命令行可执行文件等。但刚才提到的 UI 能力并不能服务于此，毕竟这是不需要 UI 能力的。

## playground & repl

对于学习 Swift 来说，playground 是一个不错的学习乐园。
playground 有两种，一种是 iPad 和 Mac 上都适配的 app，名字叫 [Swift Playground](https://apps.apple.com/us/app/swift-playgrounds/id1496833156?mt=12)，通过 SwiftUI 开发项目，可以直接上 App Store。
还有一种是通过 xcode - new - playground 生成的后缀名为 **.playground** 的项目，主要用于测试一些代码。

第一种完全是鸡肋，我是这么认为。第二种，也倾向于鸡肋，用处不大（更多用于调试，可 xcode 里创建一个项目已经足够方便）。
这里建议，**对于 Swift 研发同学来说，可以不用看 playground 了**，这样可以避免理解不少概念，如 `PlaygroundSupport`、`Sources`、`Resources`、`多target分隔`等。
这些概念和实践，在研发历程上，基本是无用的。凭空增加了 Swift 的知识点复杂度。

和 playground 类似的，还有 repl，即在终端通过 `swift repl` 进入的交互式编程环境。很多语言都带有这个环境，快速调试无可厚非，但我还是给予鸡肋的标签。

当然，对于其他语言使用者，仅仅希望了解一下 Swift，playground 和 repl 都是不错的选择，尤其 repl。可以不用理解完整项目里面的很多概念，如 AppDelegate/AppScene/Navigation 等。

## !!! Package

Apple 在 Swift 出来的时候，提出了 SPM(Swift Package Manager) 概念，和 Cocoapods 一致，做为三方库的管理工具。其中描述文件就是 `Package.swift` 文件。
后期，SPM 扩展了能力，和 `.xcodeproj` 处于同样的地位。通过对 `Package.swift` 文件进行配置，Package 所在的文件夹可以做为 `动静态库` 被 workspace 管理，也可以设置多个 Target 进行依赖控制，还可以引入三方库，甚至可以直接配置成可执行文件和插件(如代码格式化等)。
这使得对 Package 的理解复杂度增加。

一开始，Package 对 target、exec 的支持，是通过 `swift package generate-xcodeproj` 命令，生成辅助 `.xcodeproj` 文件来完成的。后期 Swift 升级，删除了这个命令，完全通过 `Package.swift` 配置文件来完成了。
这一改变，应该是为 `Swift Playground` 这个 app 做的。如上面说到的，这个 app 可以独立制作项目并上传 App Store，通过该 app 生成的项目文件是 `.swiftpm`，不用理解 `.xcodeproj - build setting` 等众多概念，多 target、三方库 均通过 `Package.swift` 配置。
可能因为这个原因，提升了 `Package.swift` 的级别，和 `.xoceproj` 处于同样的地位。

所以，对 `.xcodeproj` 熟悉的同学，不用刻意理解 `Package.swift` 里面的诸多配置，这些配置仅仅是对 `.xcodeproj` 配置的迁移。用到哪里，查一下资料即可。
下面给示例：
```
// swift-tools-version: 5.9
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
  name: "pcd", // 该 Package 的名字，如果是动静态库，可以理解为 pcd.framework。外部通过 `import pcd` 来引入
  products: [ // Build 和 Archive 输出的产物
    .executable(name: "pcd", targets: ["pcd"]), // 输出可执行文件(脚本工具等)
    .library(name: "pcdCore", type: .dynamic, targets: ["pcdCore"]) // 输出动静态库
  ],
  dependencies: [ // 三方依赖。若通过 path 进行本地依赖，则可在当前工程直接编辑其内容，便于开发联调。同 cocoapods。
    .package(url: "https://github.com/apple/swift-argument-parser.git", from: "1.2.0"),
    .package(url: "../Progress.swift", branch: "master"),
    .package(path: "/Users/gebiwanger/Downloads/swift_public_project/SwiftShell") // 本地依赖，可在当前工程直接修改库代码
  ],
  targets: [
    .target(
      name: "pcdCore" // 同 `.xcodeproj` 里面的 target 设置，这里设置一个动静态库
    ),
    .executableTarget( // 本身是 Target，同 `.xcodeproj` 主项目，可通过 Archive 进行打包（产物类似 a.out，非 app/ipa/dmg）
      name: "pcd",
      dependencies: [ // 多 target 依赖。可以依赖当前项目的 target，也可以依赖三方库(同 Cocoapods)。注意，三方库一定需要依赖，才能在当前项目中被使用(Pods 也是一样，只是Pods工具帮忙做了)。
        .target(name: "pcdCore"),
        .product(name: "ArgumentParser", package: "swift-argument-parser"),
        .product(name: "Progress", package: "Progress.swift"),
        .product(name: "SwiftShell", package: "SwiftShell"),
      ]
    ),
  ]
)
```

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202312131859591.png" width="30%">

上图是单独的 Package 项目示例，可以通过 Xcode 进行和 `.xcodeproj` 一致的调试。

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202312131859592.png" width="30%">

从上图可以验证，Package 项目，具有和 `.xcodeproj` 一致的功能，概念层级是和 `.xcodeproj` 一致的。

至此，Package 的概念和使用，基本已经介绍完毕，都是旧瓶装新酒。

### Package 插件

除此之外，还想介绍一个 Package 特有的能力，就是**插件**。插件是一个可执行程序，可以本地运行或做为三方库接入，但不允许上架包引用，仅允许做本地调试。（不排除后期会开放 App Store 的能力）
对于目前 Xcode 支持的扩展，有些能力是可以通过插件来完成的，比如代码格式化、代码转换等。
插件的整体开发过程，和 动静态库/可执行文件 的开发流程基本一致，通过下面的配置进行区分：
```
let package = Package(
  products: [ // 新增了 plugin 产物
    .plugin(name: "FormatSwift", targets: ["FormatSwift"]),
  ],
  targets: [
    .plugin( // 新增了 plugin Target
      name: "FormatSwift",
      capability: .command( // 权限控制，支持读取/写入项目文件。在代码格式化的时候需要该权限
        intent: .custom(
          verb: "format",
          description: "Formats Swift source files according to the Airbnb Swift Style Guide"),
        permissions: [
          .writeToPackageDirectory(reason: "Format Swift source files"),
        ]
      ),
    ),
    .executableTarget( // 可以有 exec target 或者动静态库 target，便于调试或者被 plugin 依赖。
      name: "AirbnbSwiftFormatTool",
    ),
  ])
```
通过 Archive 打包出的产物，可以做为可执行文件单独执行。其他项目也可以引入插件模块，右键主工程就可以看到插件的运行选项。

插件这一块的需求并不大，这里也不做更多介绍了。参考项目有 [airbnb-swift-format](https://github.com/airbnb/swift) ，有需要可以查阅更多相关资料。

## 生态小结

至此，Swift 的开发生态介绍的差不多了，整体和 Xcode IDE 依旧有非常深的绑定。
当然也可以通过 `swift xxx` 等命令行进行开发和调试，但对于 Mac 平台的开发人员来说，这完全是多此一举。

除此之外，前面也说到 Swift 是一门非常新也非常优秀的高级语言，其中的 协程、并发、多线程 等能力，也都是需要单独学习和掌握的。这属于语言本身，在环境篇就不再过多介绍。

# 命令行开发

这里进入核心环节，即通过 Swift 进行命令行开发的环境。
其实上面的环境篇，已经介绍了大部分命令行开发所需要的前期准备，如工程的搭建、项目依赖的设定等。

在 Objective-C 时期，也可以通过 Xcode 进行 `Command line` 的开发，通过 `new - project - macOS - Command Line Tool` 既可以建立一个工程，Archive 后即可输出可执行文件。
但 OC 终究不是为命令行而生的，有很多局限性和短板。这也导致十几年来一直都有的这个功能，更多被用于 C/C++ 的调试入口。

目前 Swift 做命令行开发，依旧可以走之前的流程建立一个 `.xcodeproj` 工程进行开发，区别点是语言选项上，勾选 Swift 即可。
同时，上面的流程也显得有些落伍，毕竟如上面介绍，Package 具有和 `.xcodeproj` 同样的层级定义，我们可以通过 Package 来创建可执行程序。
这样的一个明显的好处，是不用理解 AppleId / Build Setting 等复杂的配置。显然，做为一个通用的命令行工具，我们可以提交到 brew 或者随意分发给有需要的朋友，哪里还需要 AppleId 开发者账号呢？

Swift Package 命令行的项目创建/编译/调试/运行，可以通过 `swift xx` 命令行操作。有了 Xcode 这个高度集成的工具，显然不用如此费力。
创建项目的流程为：new - Package - macOS - Command Line Tool / Executable。
这样创建的工程就是 Package 工程，而不再是 `.xcodeproj` 工程。`Command Line Tool` 和 `Executable` 基本没有区别，前者多了一个 `ArgumentParser` 这个 apple 自带的库依赖。也正因为这个依赖，推荐使用前者，这在命令行开发的过程中会方便很多。

## 命令行输入参数解析自动化

可以通过 CommandLine 模块做参数解析，提供了 argc 和 arguments 等 api，可以获取用户在执行命令的时候携带的参数。

不过后面，Apple 提供了 [ArgumentParser](https://github.com/apple/swift-argument-parser) 模块，这是可以快速搭建 命令行参数定义 的框架，比如命令中需要用户携带文件路径，通过`@Argument(help: "Please Input File Path.") var path: String` 这样的定义，就可以指定用户一定要传文件路径并会被赋值到 path 变量中。如果用户输入错误或者使用 `xx --help`，`Please Input File Path.` 这个注释也会输出到命令行中以提醒用户。
当然，`ArgumentParser` 模块的能力远远不止这些。但可以看出来，`ArgumentParser` 定义了一套命令行参数的标准，开发人员不用再通过 `CommandLine` 模块调用 api 进行解析。
更重要的是，`ArgumentParser` 提供了一套命令行开发的流程框架，这可以通过阅读官方的示例文档进行学习。

前面说到 `Command Line Tool` 和 `Executable` 两种方式创建的项目相比，前者多了 `ArgumentParser` 引入。这也是推荐使用前者的原因。

当然，对于 `ArgumentParser` 的使用，可以参考官方的示例 (Examples 文件夹)。也可以查阅更多相关资料。

## 善用 Xcode IDE 的调试技巧

命令行开发过程中，一样需要断点调试，同时也需要真实环境下调试。这两者往往是同时进行的。
这里有一个小技巧，即通过在 Xcode Build 过程中插入脚本，和 Xcode Run 过程中插入参数，来完成快速的开发调试。

### 插入脚本

可以在 Scheme - Build - Post actions 中嵌入 bash 脚本，从而将编译好的可执行文件自动复制到 `/usr/local/bin` 目录，这样写完代码 Build 完成后，既可以在终端中进行真实场景下的调试和测试。
值得注意的是，一定要将 `Provide build settings from` 选择为当前编译的项目，这样才可以使用 Xcode 预定义的脚本变量。 

```
cp "$TARGET_BUILD_DIR/pcd" "/usr/local/bin"
echo "move $TARGET_BUILD_DIR/pcd to /usr/local/bin success."
```

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202312132012823.png" width="30%">

### 插入参数

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202312132019237.png" width="30%">

## 进度条的输出

进度条输出，和以往的 print 不太一样。因为 print 是叠加的，下一条输出和上一条会间隔一行。
而进度条的输出，是重写上一行。
这里需要一些小技巧，即：不换行更新上一次的输出结果

### 方案1（不建议）

```
while true:
  print("Count: \(counter)", terminator: "\r")
  fflush(stdout)
```

通过 `\r` 将当前输出的 seek 切换到当前输出行的行首。这样下一次输出的时候，就从当前行的行首输出，对上一次的输出进行覆盖。
因为没有回车符了，而终端是识别到回车符，然后将回车符之前缓冲区的内容一次性进行输出的。
所以需要使用 `fflush(stdout)` 进行强制缓冲区刷新，即强制输出。

但上面介绍的这种方式有个缺点，即覆盖内容只能按照字节长度进行覆盖。如上一次输出是 `abc`，下一次输出是 `12`，此时显示的内容是 `12c`。
这在进度条中不容易出现，但面对复杂场景显然是不行的。

### 方案2（建议）

这里推荐第二种方式，可以有效解决这个问题：
```
while true:
  print("\u{1B}[1A\u{1B}[K\(progressBar.value)")
```
输出文本加上 `\u{1B}[1A\u{1B}[K` 前缀即可。技术方案和上面是一样的，但可以规避上面的问题。
不过这里有个小 case，即单纯这样使用，会发现第一次输出的时候，整个命令行已有的内容都消失了。所以还需要加一个补丁：
```
// the cursor is moved up before printing the progress bar.
// have to move the cursor down one line initially.
print("")
while true:
  print("\u{1B}[1A\u{1B}[K\(progressBar.value)")
```

在进行输出的时候，先输出一个空内容，这样就不会有任何异常了。

## 异步等待

命令行执行完毕，进程就销毁了。如果需要进行网络请求等异步操作，就需要特别处理，以等待异步请求的结果。

这里可以通过 runloop 将当前线程永久激活，然后通过 exit 强制退出。
```
async {
  ...
  exit(0/-1)
}

RunLoop.current.run()
```

显然，这不是一个好的方案，一来局限性很多。
在者，runloop 是 app/ipa/dmg 开发中的概念，虽然利用了 Swift api 完备性的优势，但在这里使用很不合适。

这里更好的解决方案是使用信号量：
```
let semaphore = DispatchSemaphore(value: 0)
async {
  ...
  semaphore.signal()
}

semaphore.wait()
```
可以将信号量的操作，进行单独封装，后续需要 wait 的时候，只需要便捷的调用一些 api 即可。

当然，鉴于 GCD 强大的多线程管理能力，还可以使用 group 等方式，进行多个异步操作的并行操作等。

## 系统命令

当然，命令行开发过程中，不可避免需要使用到丰富的系统命令，如 `ls/grep` 等。
这可以通过 Process Api 来操作，但这里不做过多介绍，因为有强大的三方库可以使用。
详见：[【Swift 三方源码1】SwiftShell 高效的命令行工具](https://www.yigegongjiang.com/2023/SwiftSystemShell/)

## 命令行小结

以上，就是通过 Swift 做命令行开发中的一些概要和部分细节的地方了。
如最开始说到的那样，使用 Swift 开发命令行，除了 UIKit/AppKit/SwiftUI 这些 UI 属性不能使用之外，其他并没有太多的 api 限制。
甚至可以使用 UserDefault 存储，这样可以极大的简化用户的 cookie 等信息的保存。

除此之外，大概就是对 Swift 高效和美丽的使用了。
Swift 这门语言虽然上手容易，但美丽的使用并不简单。
巧妙的使用 enum、protocol、async、combine 等高级功能，并且不搞乱代码，就很优秀。

___

什么都给了，又什么都没给。
什么都干了，又什么都没干。
什么都说了，又什么都没说。

放开了所有权限，又加上精细的卡口。

鸡和蛋的轮回，永不停息的魔咒。
