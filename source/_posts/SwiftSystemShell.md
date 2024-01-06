---
title: 【Swift 三方源码1】SwiftShell 高效的命令行工具
date: 2023-12-13 13:56
categories:
- 技术
tags:
- Swift
- Swift三方源码系列
---

> 不推荐使用 Swift 写脚本，和 Python 比起来，该生态链相对匮乏，开发耗时会增加很多。
> 但对于偏 Swift 的同学来说，这也的确是更可控和方便维护的方式之一。尤其对于公司内部工具，有问题可以更快的找到原因并处理，不至于手忙脚乱。

通过 Swift 进行脚本开发的环境和入门，可参考：[Swift 脚本开发环境搭建](https://www.yigegongjiang.com/2023/SwiftCommandEnv/)

在开发脚本的时候，经常会使用到在终端环境中安装/预制的命令、管道、文件读取等。这些能力，[SwiftShell](https://github.com/kareman/SwiftShell) 做了完备的封装，很方便使用。
这里对该库源码进行一些解读。同时，也会做前置知识点如文件、管道、描述符、子进程进行简单介绍。

<!-- more -->

# 前置

在进行源码分析之前，需要先对 Swift 中的 文件读取 和 进程 进行概要描述，这是 SwiftShell 重点依靠的系统能力。熟悉它们之后，可以更熟悉 SwiftShell 的 Api 到底接管和实现了哪些能力。

## 0x01 FileHandle 

Swift 中 Filehandle 属于 FileManager 这一类文件操作 api 的一部分。FileManager 一般使用场景是创建文件/文件夹，毕竟容易理解。不过 Filehandle 中有些知识点，还是先熟悉一下比较有益。

### stdin/stdout/stderr

这三者即标准输入流/输出流/错误输出流，经常会听到，使用也比较简单，但若没有使用过，还是有些雾里探花。
这三个标准流，一般不需要设置，操作系统或者运行时环境会在启动应用程序的时候，默认进行设置。
其中，stdin 默认是键盘输入，stdout 默认是屏幕，stderr 默认也是屏幕。所以，使用 print 进行打印的时候会在电脑屏幕上进行展示，也可以在终端里面输入一些字符以和程序进行互动响应。
对于命令行可执行文件，当然依旧遵从该法则。
但是对于手机或者电脑里面的应用程序，默认就会关闭 stdout 和 stderr 了，因为这个时候已经离开调试环境，print 输出是没有必要的，更需要的是[日志](https://www.yigegongjiang.com/2023/track/)，而这需要磁盘存储。
当然，也可以在应用程序启动后，手动更改 stdout 的指向，从屏幕改为文件，这样就可以将 print 的内容输出到指定文件夹，可以这样设置：`freopen(file,"a+",stdout)`

对于系统的设计，有一个规则即**一切皆文件**，比如内存、键盘、缓存、网络 Socket 等。即只要是有字节输入输出场景的，都可以通过高层级的**文件**进行抽象和描述。
stdin/stdout/stderr 也不例外，实际上它们就是三个文件，文件描述符分别是 0/1/2。
下面通过不同的编程层级进行描述，详见注释：
```
底层: unistd.h
#define   STDIN_FILENO  0  /* standard input file descriptor */
#define  STDOUT_FILENO  1  /* standard output file descriptor */
#define  STDERR_FILENO  2  /* standard error file descriptor */
> 这是底层 POSIX 接口，进行底层开发的时候会用到上面三个宏定义，根据后面的注释可知，0/1/2 是三个基础输入输出流的文件描述符，其他描述符不会暂用。 

C 运行时：stdio.h
extern FILE *__stdinp;
extern FILE *__stdoutp;
extern FILE *__stderrp;

#define  stdin  __stdinp
#define  stdout  __stdoutp
#define  stderr  __stderrp
> 这是 C 语言(高级语言)运行时提供的接口，进行 C 语言编写或者高层级语言编写的时候可以使用。这里把基础输入输出抽象为文件了，通过文件指针进行定义。

Objective-C：NSFileHandle.h
@property (class, readonly, strong) NSFileHandle *fileHandleWithStandardInput;
@property (class, readonly, strong) NSFileHandle *fileHandleWithStandardOutput;
@property (class, readonly, strong) NSFileHandle *fileHandleWithStandardError;
@property (class, readonly, strong) NSFileHandle *fileHandleWithNullDevice;
> 这是 OC 语言 (高级语言) Foundation 库提供的接口，更高层级的开发过程可以使用，如应用、app开发。这里做了更高层级的抽象，即抽象为 FileHandle 对象。

Swift：FileHandle
open class var standardInput: FileHandle { get }
open class var standardOutput: FileHandle { get }
open class var standardError: FileHandle { get }
open class var nullDevice: FileHandle { get }
> Swift 的 FileHandle 是对 OC 的桥接，所以基本一致。
```

这里有个小小的剧透，在可执行文件中执行终端命令的时候，默认是有输出的（可执行文件在终端运行，输出默认显示在屏幕/终端）。
而这可能不是我们想要的，因为我们可能**希望拿到命令执行的结果**或者**根本不在乎命令执行的过程**。
这个时候，可以在启动终端命令子进行的时候，手动设置其输出属性。
前者，可以提供一个自定义的 FileHandle 对象(内存数据抽象的文件或者真实磁盘文件都可以)，这样终端命令的结果就会输出到自定义的对象中，而后从该对象取数据即可。
后者，可以提供 nullDevice，将不会有任何输出。

当然，这里还有一个小提醒，线上程序最好是使用日志记录代替 print。这种场景下，操作系统虽然没有为输出设置为 stdout，即不会有显示，但 print 的用户态代码还是会执行的，只是到了内核态会找不到 stdout，不做输出而已。这会有性能损耗。

> 这里对 stdout 和 stderr 做下区别，即 stdout 用作输出，stderr 用作错误输出，这是不同的。
> 苹果平台用了很多年的 NSLog(ASL日志系统) 就没有准确认清这两者的概念，NSLog 是使用 stderr 做输出的，不会走到 stdout。
> 这是不对的。后面改用了 oslog，但为时已晚，NSLog 已经铺天盖地了。所幸在 Swift 里面停用了 NSLog 的桥接。

### 文件读取和文件指针

FileHandle 类是对文件进行面向对象的描述，操作文件应该通过该对象进行。当然各个高级语言均有对应的操作 api，这里通过 swift/OC 的 Filehandle 进行介绍。
文件有最基础的三个访问属性：可读/可写/可读写，在 FileHandle 里分别对应三个初始化函数。（除此之外，FileHandle 也可以通过文件描述符进行初始化，文件描述符是一个文件的访问替身，其本身定义的时候，也需要设置这三个属性）

在成功访问了文件之后，还有一个重要的属性，即 seek(文件指针)，它表示当前正在操作文件的哪个位置。初始化 FileHandle 对象后，默认位置是 0，即文件开头。
seek 在文件处理过程中非常重要，很多 api 的访问会默认修改 seek 的值。对于追加文件内容的场景，可能追加在不同的位置，这里就需要自行控制 seek。

对于可读场景，可以通过 `readDataToEndOfFile` api 直接读取整个文件的内容，过后 seek 会被修改为文件尾。
也可以通过 `readData(length)` api 读取部分内容，过后 seek 会做 length 大小的 增偏移。
之所以可以循环调用 `readData(length)` api 直至读取全部内容，是因为内部 seek 值一直在变化，这样 readData api 才知道从哪里开始读。
`readLine` 也是同理。

看起来，可读场景下开发人员对 seek 并不感冒，因为 api 内部做了很多工作。但是可写场景就不一样了。 
对于一个有内容的文件，如果要追加内容，必须注意是否及时调用了 `seekToEndOfFile` api，提前将 seek 指向文件末尾。不然写的内容就可能错位了。
这里给一个该场景需要注意的例子，对于可读写的文件同时做读和写的操作，就需要时刻关注 seek 所处的位置，详见注释：
```
// 这里通过文件描述符打开一个文件，设置可读可写属性 O_RDWR
let fileDesc = open(path!, O_RDWR)
let fileHandle = FileHandle(fileDescriptor: fileDesc,closeOnDealloc: true)

// 下面通过 filehandle 对文件进行读取，追加写入后立刻从文件头按字节读数据
for i in 0...1000000 {
  // 写操作
  try fileHandle.seek(toOffset: fileHandle.seekToEnd()) // 将 seek 移动到文件尾，以在尾部追加当前 i 到值（1/12/123/1234/...）
  fileHandle.write(String(i))
  Thread.sleep(forTimeInterval: 0.5)
  // 读操作
  try fileHandle.seek(toOffset: 0) // 将 seek 移动到文件头
  print(fileHandle.readData(ofLength: 2 * i)) // 每次循环从文件头开始多读取 2 个字节。（1/12/123/1234/...）
  Thread.sleep(forTimeInterval: 0.5)
}
```

通过一个 FileHandle 对象对文件进行读&写操作，还是有一定风险的。毕竟上面的示例只是最基础场景的应用。
所以一般都不这么做，而是对同一个文件设置 read 和 write 两个 fileHandle 对象，这样就可以分别操作互不影响。
通过 `tail -f` 查看实时日志就是这种场景，即一个 handle 只负责追加内容到文本。另一个 handle 只负责读取。
这个时候，有一个便利的 api，即 `availableData`。它可以读取 seek 至文件尾的内容，并自动将 seek 调整至文件尾。
这样只需要定时的访问 `availableData`，就可以实时读取到另一个 fileHandle 写入的新增内容。
当然，如果需要监听者模式的响应，可以不使用定时，使用 FileHandle 的 `readabilityHandler: (@Sendable (FileHandle) -> Void)?` api。
当文件被追加写入后，该回调就会被调用。这个时候在调用 `availableData` 拿数据，就非常顺理成章了。

#### Pipe 管道

对于通过两个 Filehandle 对一个文件分别进行读写的例子，就是 `Pipe` 管道。管道的定义很简单：
```
open class Pipe : NSObject, @unchecked Sendable {
  open var fileHandleForReading: FileHandle { get }
  open var fileHandleForWriting: FileHandle { get }
}
```
提供了两个 FileHandle，这两个 FileHandle 返回的并不是同一个可读可写 handle，而分别是读 handle 和写 handle。
通过上面的介绍，应该对 Pipe(管道) 的作用更加清楚了吧。
就是 fileHandleForWriting 写入的内容，可以从 fileHandleForReading 读出，像一根管子的两端，实际上是文件的抽象，仅此而已。

## 0x02 Process(进程)

在 [Shell 和进程](https://www.yigegongjiang.com/2022/shell/) 章节中，详细的介绍了进程和 Shell 的关系。
在可执行文件中调用终端命令，就需要创建子进程，并在子进程中运行命令，而后跨进程通信以拿到返回值。
基本原理详见 `Shell 和进程` 一文，这里仅对 Process 模块 api 进行介绍。

对于使用 OC 进行移动开发的同学来说，可能没有使用过 Process 模块，毕竟 iPhone 是不允许多进程的。甚至小组件等能力，也通过新的独立 app(扩展) 来实现，而不是在主应用中开放子进程接口。
简单来说，Process 是 Swift 进行多进程编程的实现，但是是阉割版。
实际上，Swift 依旧只提供了 GCD/Combine 这样的多线程能力，而没有开放多进程能力，即可以随意创建子进程并在子进程中执行任务。
Process 简单来说仅仅是为终端命令的执行提供了场所环境。其 api 也完全对应 终端命令 的参数格式。

接口比较简单，如下：
```
executableURL(同废弃字段 launchPath)：终端命令的完整路径，如`/bin/pwd`,`/bin/ls`。
arguments：携带参数，如`-t`，`-v`，`--help`。
run(同废弃字段 launch)：启动 process，开始执行命令。
waitUntilExit：和信号量的 wait 一样，等待命令返回。
terminationHandler：是 waitUntilExit 的监听者模式的实现。通过回调告知命令执行状态。
terminationStatus：当前命令的返回值，默认正常，即 0。如果命令返回 error，一般使用 -1 等非 0 值。
standardInput/standardOutput/standardOutput：前面介绍的三个标准输入输出及错误流。这里是从命令的视角来看的，即这三个字段的赋值代表数据的流向。可以为 standardOutput 指定一个磁盘文件，这样命令的输出就会写入文件。
environment：环境变量，可以读取父进程的 env后，携带到子进程。在 `Shell 和进程` 一文中有介绍。
suspend/resume/isRunning：意如其名
```

这里重点说下 executableURL(launchPath)，最好是提供命令的完整路径，如 `/bin/pwd`。完整路径可以通过先执行 `/usr/bin/which pwd` 命令来拿到。
如果想简单使用，可以仅提供 `pwd` 也一样可以，默认会从系统的环境变量中查找。
但这有一定的风险，因为除了系统命令，还有用户自己安装的其他命令，被放置在各个位置。且 Swift 是跨平台的，部分系统用户还可能会自定义系统路径的位置，可能会出现找不到命令的情况。

还有一个办法是将 executableURL 设置为 `/usr/bin/env`，然后将 `pwd` 写入 arguments 参数中。
这样的话，默认会读取 `env` 里面的 PATH 以获取 `pwd` 的完整路径。而 `env` 中 PATH 基本是全的。

# 源码分析

SwiftShell 源码已经停止更新较久了，部分 api 已经不符合当前 Swift(5.9.2) 的标准建议。
但 SwiftShell 本身是一套非常精妙小巧的框架设计，可以快速在项目中进行改写并使用。实际上，目前 Swift 5.9.2 版本使用，并不需要做调整，即可正常使用。
如需要进一步调优以适用于自己的项目，可在查阅原作者源码或下文分析后，自行修改。

源码分析注释已经同步更新于 Github 项目中，可直接阅读该项目：[SwiftShell 源码解析](https://github.com/yigegongjiang/SwiftShell)

**前置** 中已经对 FileHandle、stdin/stdout、file seek、Pipe、Process 等做了介绍。了解了这些后，可以比较方便的阅读源码，注释中也对必要的环境进行了阐述。
还有一些 SwiftShell 中用到的知识点，这里也做一些说明，或许可以更利于理解项目：

## 终端 env 参数传递

打开终端解释器后，bash/zsh 等环境初始化成功后，当前进程会有一些已经配置好的环境变量。
这些环境变量有些是需要传递给通过 SwiftShell 执行的子进程命令的。
具体的操作流程并不复杂，但是对于环境变量是如何传递给子进程的，以及子进程对变量的操作是否会影响到父进程，可以查看 [Shell 和进程](https://www.yigegongjiang.com/2022/shell/#0x03-Shell-%E5%92%8C-SubShell) 以获取更详细的说明。

## Lazy 延迟计算

鉴于 FileHandle 为文本 IO 操作，SwiftShell 内部为防止耗时，已经尽可能的使用 lazy 延迟迭代器对 IO 数据进行解析。

Swift 中对 Sequence 序列进行 Lazy 操作，是通过在内部新建一个内部类迭代器完成的。

```
// 自定义 `CycleIndex` 迭代器，同时实现 `Sequence` 协议，作为序列使用 lazy 操作及高阶操作
class CycleIndex: IteratorProtocol, Sequence {
  typealias Element = Int
  var index: Int = 0
  func next() -> Int? {
    defer {
      index += 1
    }
    guard index < 10 else {
      return nil
    }
    return index
  }
}
```

```
1. var i = CycleIndex().lazy
2. var j = i.map { pass }

3. for _ in j {} OR j.makeIterator().next()
```

第 1 步，通过 lazy 计算属性会根据 CycleIndex() 对象生成一份新的对象 `LazySequence` i。

第 2 步，对 i 进行 map 访问，会继续生成一个 `LazyMapSequence` 对象 j 并将 map 闭包存储于对象 j 中，这就是 lazy map 不会立刻执行的原因。
在 j 内部有一个内部类 `Iterator` **n**。该内部类实现了 `IteratorProtocol` 协议，即 **n** 是一个可迭代对象。

第 3 步通过 for...in 或 next 对 j 进行访问的时候，就会调用到 **n** 的 next() 方法，该 next() 会对 j 持有的 map 闭包进行执行。外部每调用一次 next()，map 闭包就会执行一次。

这是 lazy 能够对 序列 进行延迟访问的原因。

```
// lazy 计算属性实现
public var lazy: LazySequence<Self> {
  return LazySequence(_base: self)
}

// 可迭代内部类设计
extension LazyMapSequence {
  public struct Iterator {
    ...
  }
}
extension LazyMapSequence.Iterator: IteratorProtocol {
  public mutating func next() -> Element? {
    return _base.next().map(_transform)
  }
}
```

## 类型檫除

对于范型协议，是不能直接用作返回值的，如：
```
protocol Fly {
  associatedtype action
  func canFly() -> action
}
}

class Cat: Fly {
  typealias action = Bool
  func canFly() -> action {
    false
  }
}

class Bird: Fly {
  typealias action = String
  func canFly() -> action {
    "flase"
  }
}
```

以上定义了 Fly 协议，两个实现类中对于是否可以 Fly 的返回类型是不一样的。

```
func theAnimal() -> Fly {
  ... {
    return Cat()
  }
  ... {
    return Bird()
  }
}
```

函数返回 Fly 协议，Xcode 检查会不通过，编译报错。因为当前返回值具有多样性，Xcode 并不能知道具体的返回类型是什么。
比如：`let m = theAnimal().canFly()`，此时 m 是 Bool 类型还是 String 类型，就无法判断。

私认为这是 Swift 对 protocol 实现范型 不友好 的地方，官方没有提供友好的解决方案，而问题又的确是需要解决。
公认的解决方案是：类型檫除。使用中间层代理的形式，以 Anyxxx(AnyIterator、AnySequence) 对原有的数据进行中间层封装。

Swift 内部也在大量使用该方案，可查阅 Swift 源码 `ExistentialCollection.swift - AnyIterator`。源码可参考：[Swift 官方源码 Lite](https://www.yigegongjiang.com/2023/SwiftDependXcode/#0x02-Swift-%E5%AE%98%E6%96%B9%E6%BA%90%E7%A0%81)

在 `AnyIterator` 中，定义了 `_AnyIteratorBoxBase`、`_IteratorBox` 两个关键类，用于明确 Iterator 的 next() 返回值到底是什么类型。
这样可以在任意位置正常使用 `AnyIterator`，因为通过范型参数的形式，Xcode 已经明确知道迭代值是什么。 

理想的类型檫除，是 Xcode 明确知道类型是什么，可以使用静态调用，代码执行会更高效。
后期 Swift 的确推出了较友好的一个方案用于类型檫除，即 **some**。用于 Xcode 对返回类型进行智能推断，很多场景下均可以使用 **some** 做类型檫除。

除 **some** 外，还有 **any** 也可以用于类型檫除。使用 **any** 后，Xcode 就完全不管具体类型，而是交由运行时判断。这相对而言是有一定风险和性能损耗的。
鉴于“又不是不能用”原则，在降低代码复杂度、可维护等理由下，若 **some** 无法满足，还是推荐 **any** 的。
准确来说，如果项目中需要考虑到 **any** 的性能损耗，那一定是还有其他更需要解决的问题。

但对于开源的项目，还是建议使用公认的`类型檫除`方案，即 Swift 官方也在大量使用的 中间层 代理方案。
虽然略显复杂，但却十分行之有效。

___

荣耀归于主，因祂配得我们一切的赞美和敬拜。

