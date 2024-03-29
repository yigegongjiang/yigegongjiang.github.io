---
title: 移动端日志系统怎么搭建
date: 2023-01-23 02:34:25
categories:
- 技术
tags:
- C
- iOS
---

端上日志系统非常重要，对于用户侧的异常、排障、动线、行为等很多重要数据，都可以通过端上日志来做检索。如何搭建一套准确、高性能的完备日志 SDK 就显得尤为重要。 （另一个重要的排障信息源是埋点，通过埋点可以获取更精准的用户动线。后面有时间做一下埋点数据化方面的总结）。
移动端日志系统，将承载 Native、h5、动态化等多技术栈环境下的日志收口工作，同时要兼顾日志不丢不乱和高性能，其实还是有不少挑战的。
这一方面**CocoaLumberjack**其实已经做的很好，很多公司都用它作为自己的日志系统的基础框架。但它还不能作为大型 app 的流量日志收口系统。因为流量大了以后，少量的日志丢失也会带来很大的缺口，而性能方面它也有很多短板。
**mmap** 可以在 IO 性能方面有显著的提升，也就是后端比较通用的**零拷贝**技术。在移动端上**FastImageCache**对 mmap 有较深的应用，但它业务绑定太强，一般无法直接使用，更多的是学习 mmap 的落地。

本文会对日志系统的一些完备要素做一些说明，并特别讲解下**CocoaLumberjack**、**FastImageCache**两个技术库。
* 日志是否全量。很多业务开发同学不使用日志 SDK 做日志输出，可能使用系统日志做打印，这一部分日志是否需要做收口，需要权衡一下。
* 分等级和模块。分 Level 和 modules 进行记录和检索，可以提供问题排查速度。这方面有专业的 Debug/Info/Error 等标准。技术上比较好实现。
* 性能。主要是卡顿和耗电。日志系统会底层基础架构，大流量的打入，会频繁的内存释放、I/O，过多的占用 CPU 会导致卡顿和耗电，也会影响拖延上层业务异步代码的执行时机。
* 数据不丢不乱。没有办法做到完全的不丢日志，只能尽可能少的减少。在 crash / CPU 繁忙 / 压缩加密严重耗时拖延队列等异常场景，这方面问题尤其突出。不乱就需要增加串型队列或者锁，这些同步机制都需要保障阻塞和性能。
* 实时观测。开发和测试同学需要能够实时看到日志打印情况，这在开发和提测阶段非常有用。
* 压缩+加密。压缩和加密都是耗时操作，对 CPU 的压力比较大。
* 上报。得有回捞机制，得保障数据传输安全。
* 后端系统。前置的采集完成后，后端系统的数据化检索、数据可视化等工作都是重中之重。
* 隐私安全。这一块国内所有厂商都极度匮乏。大厂的开发同学也可以随意捞取用户日志，异常日志在内部系统可以随意传播。如果有这方面的诉求，那么整套安全体系都需要建立起来。

<!--more -->

## 全量日志的收集

端上打日志的方式有很多，尤其在日志系统没有搭建之前，每个开发同学可能搭建了自己的小型日志记录框架或者系统特有的 log api。如果业务有需要，这部分日志 SDK 之外的流量是可以收口的。
对于 iOS 来说，比较常见的就是 nslog、printf、os_log，少见的是 write，NSFileHandle，syslog，nslogv。其中 nslog 场景最多，需要特别处理。
下面分别对这些系统日志系统做一些技术说明。

### 日志接口说明

* stdin/stdout/stderr

> 2023.12.14 更：[【Swift 三方源码1】SwiftShell 高效的命令行工具](https://www.yigegongjiang.com/2023/SwiftSystemShell/#0x01-FileHandle) 有更完善的描述

这是三个默认的 Unix 系统输入输出终端（终端也是文件）。printf 使用的 stdout 输出，即标准输出。而 nslog，苹果对其定义是异常和错误记录，使用的是 stderr 输出。
所以如果想屏蔽/重定向 xcode 的日志输出，printf 和 nslog 需要分别使用 `freopen(file,"a+",stdout);` 和 `freopen(file,"a+",stderr);` 进行输出重定向。

```
// unistd.h

#define   STDIN_FILENO  0  /* standard input file descriptor */
#define  STDOUT_FILENO  1  /* standard output file descriptor */
#define  STDERR_FILENO  2  /* standard error file descriptor */
```

* printf、write
printf 是终端打印 api，默认做 stdout 标准输出打印。如果没有终端，就不会有打印了。write 是文件写入 api，可以将文本、二进制等数据写入到指定的文件中。
printf 底层是用 write 实现的，即 write 的指定文件是 stdout，即终端。

```
// uio.h

struct iovec v[13];
// 拼装 v
v[0].iov_base = "testLog:";
v[0].iov_len = 8;

v[1].iov_base = (char *)msg;
v[1].iov_len = msgLength;

// 输出到 stderr 终端，模仿 nslog
writev(STDERR_FILENO, v, 2);
// 输出到 stdout 终端，模仿 printf
writev(STDOUT_FILENO, v, 2);
// 输出到文件
writev(file, v, 2);
```

这里有个注意的地方，如果没有终端，如真机运行的时候，虽然不会打印日志，但是 printf 的代码执行流程不会停止。
因为有重定向等操作，在调用 printf 的时候，是不知道最后到底要不要输出的，这个时候还处于用户态。所以 printf 的所有代码都会执行一遍，最后进行系统调用。
系统调用的时候，才知道输出到哪里，如果没有终端那就不输出。
所以对于 release 线上包，即使 printf 最终没有终端来打印，但用户态的代码逻辑都会被执行。这其实是无用的损耗。  

* syslog

syslog 是 Unix 系统下的常用日志系统，iOS 通过引入 `sys/syslog.h` 也可以使用。
syslog 会将信息打印到终端，并保存到`var/log/syslog`文件中，还可以将日志文件进行压缩，以及实时网络传输功能。syslog 是一整套日志服务工具，跨平台使用非常棒。
直观来看，可以认为 syslog 是 printf 的大号升级版。printf 只能做终端输出，syslog 还可以做文件存储等工作。syslog 是跨平台的日志方案，可以进行多样的配置以满足多样需求。

```
// sys/syslog.h

openlog("info",LOG_PID,LOG_LOCAL5);
syslog(LOG_INFO, "hello %s","woring");
closelog();
```

* nslog、nslogv、asl、oslog

nslog 和 syslog 在同一个级别，对于自家系统，苹果可能觉得 syslog 太大或者性能等综合考虑，重新做了一套 ASL 系统，后面又做了一套 oslog 系统。

ASL 主要做了两件事, oslog 也有同样的功能。
1. 输出日志到 asl 系统。asl 系统会对日志做文件存储，也可以通过 mac 的 console.app 软件进行日志的实时查看、检索、过滤。
2. 输出日志到 xcode 控制台

在 iOS 10 之前，苹果使用的是 ASL 系统。后面觉得性能有待提升以及跨平台，升级到了 oslog。
在 CocoaLumberjack 中作者是这么描述 nslog 和 ASL 的：
```
// nslog
 * the traditional NSLog() function directs its output to two places:
 *
 * - Apple System Log
 * - StdErr (if stderr is a TTY) so log statements show up in Xcode console

// asl
NSLog does 2 things: 
  * It writes log messages to the Apple System Logging (asl) facility. This allows log messages to show up in Console.app. 
  * It also checks to see if the application’s stderr stream is going to a terminal (such as when the application is being run via Xcode). If so it writes the log message to stderr (so that it shows up in the Xcode console).
To send a log message to the ASL facility, you basically open a client connection to the ASL daemon and send the message. BUT - each thread must use a separate client connection. So, to be thread safe, every time NSLog is called it opens a new asl client connection, sends the message, and then closes the connection. 
NSLog 会向 ASL 发送日志信息，会出现在Console.app 中。同时向 Terminal 发送日志信息。
并且每一次 NSLog 的输出，都会新建一个ASL client 并向 ASL 守护进程发起连接，发送日志信息之后再关闭连接。
```
oslog 会做一些内存和磁盘的优化，具体没有看，性能肯定有一些提升。而且做到了多平台，在苹果全家桶上面都可以用了。

nslog 在 iOS 10 之前使用的是 asl，之后也迁移到了 oslog 了。nslog 是通过 nslogv 实现的，它们没有太多区别。
nslog 对于终端的输出，最终会通过 fprintf 实现，`fprintf(stderr, "%s\n", buf);`，最后也是和 printf 一样通过 `write` 对 stderr 进行写入。
nslog 对 asl 的操作，是通过 `asl.h` 接口实现的。
```
#import <asl.h>

aslclient _client = asl_open(NULL, "com.apple.console", 0);
aslmsg m = asl_new(ASL_TYPE_MSG);
asl_set(m, ASL_KEY_MSG, "msg")
asl_send(_client, m);
asl_free(m);
```

### 如何捕获全量日志

其实最全的捕获方式，就是 hook write 接口。因为这是 C 对于文件写入的用户态底层接口，只要是日志都会走到这里。通过**fishhook**可以做进一步的捕获。
不过这个接口是文件写入接口，除了日志，其他写操作也会走到这里，会有很大的数据干扰。最好还是在上层进行流量捕获。
printf 的捕获，可以直接 hook，没有太多办法。
基于 ASL 的 nslog，可以通过系统 api 进行捕获，在 **CocoaLumberjack** 的 **DDASLLogCapture.{h/m}** 文件中有详细说明，可以借鉴。
基于 oslog 的 nslog，就没有系统 api 了。可以直接 hook nslog/nslogv 来捕获。建议 hook nslogv，因为有些业务同学可能自己搭建了一套小型的业务日志系统，会直接使用 nslogv。如果 hook nslog，会有遗漏。
对于 nslog 的捕获，也有 stderr 重定向(dup2,pipe)等方式，但感觉没有 hook 来的方便，因为重定向需要操作文本，pipe 需要操作数据管道，实时性、性能、复杂度都比不上直接 hook。
对于 os\_log 的专有日志 api，也可以 hook。
所以捕获全量日志，就靠**fishhook**了，它依靠动态库的函数符号三级跳的特性，改写符号地址来实现 hook，本身非常小巧。但是 fishhook 只能 hook 动态库的函数符号，自己写的一些 C 函数是 hook 不了的。

## 日志系统的流量如何实时观测
开发和测试同学需要能够实时看到日志打印情况，这在开发和提测阶段非常有用。业务多了之后日志会大量刷屏，一定要对观测数据做行过滤。
所以日志的实时观测有这三个强诉求：
1. 打印内容足够多。xcode 和 console.app 会对内容做剪裁，超过一定长度大小后的内容就不展示了，这个需要处理。
2. 支持行过滤，需要对字典/数组等容器对象做序列化，将内容拼接成一行文本。
3. 内容不乱。xcode 的终端输出，会出现文本相互嵌套乱序的情况。主要是因为输出缓冲区的缘故，容器内容输出有太多换行符。将每个日志拼接成一行文本可以解决这个问题。

### 打印内容足够多
通过 nslog/os\_log 输出，xcode 和 console.app 会有字符长度限制。通过 printf 输出可以突破这个限制，但是 printf 前面说过只会输出到 stdout，并不会输出到 console.app，对于测试同学就看不到日志了。
单独使用 nslog/printf 是无法同时满足开发和测试同学的诉求的。
可以在 ide 环境将日志做 printf 输出，先满足开发同学的全量数据观测。
提测阶段，不在 xcode 下运行，字符长度限制会更小。有两个解决方案：
1. 将长内容按照 200 长度大小做分割编号后打印（开发阶段字符长度限制大约在 800-1000）
2. 开发一个 pc 版本的在线日志工具。通过**CocoaHttpServer**这样的轮子，将端上日志通过 http/socket 实时传输到 pc 端

### 支持行过滤&内容不乱
这个没什么要多说的，主要是为了方便检索和过滤，用处非常大。

### nslog/os\_log 的使用
其实这里可以看到，nslog可以完全不用的。我们使用 nslog 主要是做调试输出，到线上后就基本不关注了。
从上面对 nslog 的解释可以看到，nslog 在苹果看来是异常/错误的日志输出，而且有很大的性能开销，不管使用 asl 还是 oslog 系统。
而日志的磁盘记录和上报，则需要专门的文件接口实现，下面会说到。
如果排除测试同学的数据观测，我们可以直接使用 printf 来实现日志打印。
但还是要注意，线上不能开 printf。前面介绍 printf 的时候说到过，虽然线上没有 stdout，但 printf 的用户态代码还是会执行的，只是内核态会找不到 stdout，不做输出而已。
nslog/os\_log 也不应该到线上，线上包是可以通过 console.app 查看 nslog 打印出来的日志的。一些调试信息很容易被检索到，会有数据安全问题。

### 终端日志轮子

**CocoaLumberjack** 的 **DDTTYLogger.{h/m}** 文件，对 xcode 的终端输出有现成的实现，可以直接使用。
它使用的是 `writev(file, v, 2);` 接口，前面有说明，write 是 nslog、printf 的底层实现。
它里面有个技巧，就是如果日志内容不多，会通过**alloca**申请栈空间做msg存储。日志程度较大的时候，才会通过**calloc**申请堆空间。

## 日志采集系统设计

**CocoaLumberjack**做的很好了，基本上自研的日志系统或者小型日志模块都会对它进行各种程度的借鉴。
**CocoaLumberjack**主要是通过一个流量入口，进行多个渠道的流量分发。
研发同学通过宏定义可以减少代码量和日志代码侵入，在 sdk 内部，做 磁盘文件/终端/数据库/网络 等多渠道的分发，通过 level 做不同层级的日志隔离。可以再加一下 modules 的区分，它里面没有实现，可以在上层实现。
**CocoaLumberjack**亮点是**支持业务侧的高度自定义**。可以对日志进行 format，在上层可以快速接入压缩层、加密层、网络层等业务逻辑。

但**CocoaLumberjack**并不是一个高效率的日志采集 sdk，因为它核心的功能**磁盘文件存储**性能不高，每条日志都会直接操作 IO，内部会频繁的进行文件排序/文件大小检查等耗时操作。
业务自定义的压缩加密等模块很可能有超时情况也会进一步增加它内部的队列堆积。
在大用户量的 app 中，上层业务非常复杂，日志流量会非常大。这些问题很可能导致数据丢失和 CPU 爆增。

## 解决不丢不乱和性能问题

**CocoaLumberjack**的性能问题产生原因就是因为日志数据直接入串型队列。虽然它内部通过子线程消费队列，但改变不了队列锁粒度太大的事实。
它内部会将每一个日志分发到所有日志处理源，各个处理源在内部队列中消费完毕后，通过 group notice 结束当前日志的处理，然后处理串型队列里面的下一个日志。
整个分发流程的锁粒度非常大，任何一个环节有耗时操作，都会造成当前任务延迟完成，造成队列任务堆积。这个时候用户退出app，就会导致数据丢失。
为了使得日志不乱，对 IO 的操作一定得是单线程的，否则日志内容会嵌套就没法看了，这个无法改变。
既然锁存在的事实无法改变，那么就需要在前置链路进一步降低锁的粒度。

1. 日志分发到每个渠道后，各个渠道可以自行消费，无需阻塞日志队列。
2. 各个分发渠道需要尽可能的使用并发队列和多线程。
3. 磁盘存储 sdk 模块可以对日志内容进行并发压缩/加密处理（流式处理），通过容器增加序列号用作最后的快速排序。也可以不排序，数据上报后让后端服务进行排序。
4. 日志数据可以先组装到内存，内存容器多线程写入，保持一个小粒度的锁防止异常。在时机满足后内存数据批量存储磁盘，减少 IO 次数。
5. 磁盘存储可以使用后端常用的 **零拷贝**/**mmap** 技术，使得用户态共享内核态的页缓存，避免多次无效数据拷贝。还可以共享系统的文件脏数据写入，不用在上层做冗余的 crash 等异常维护。

这里可以通过并发队列+多线程减少锁粒度来快速消费数据，通过内存暂存快速保存数据并减少 IO，通过 mmap 减少磁盘文件的写入耗时和异常处理。
其中有一个环节就是并发处理后的日志排序，建议是数据上报后让后端来做。毕竟前端的时间少一点，带来的价值就是非常大的。每一个优化，不都是争取减少那么一点时间么。后端只会处理上报的异常用户数据，即使增加几秒排序的时间，也是值得的。
这里的未排序并不是乱序，乱序是多个日志内容出现嵌套，这里虽然没有排序，但每条日志都是独立的，不会出现嵌套情况。

mmap 的应用，在下面会通过**FastImageCache**做单独说明。

## 回捞、上报、数据安全、其他

回捞功能必不可少，需要用户日志，基本都是回捞场景。
回捞肯定得通过 socket 来做，如果已经有 IM 功能，那么可以直接集成。如果没有实时通讯能力，依靠 push 也可以来做。
回捞的时候需要注意一下回捞哪些时间的文件，以及文件大小处理。如果文件过大，建议分批回捞，防止一次性回捞的时候每次都失败。

上报可以走统一的网络服务了。日志文件一般都会比较大，动不动就是十几 M 了。可以走 oss 服务，切片上传。

数据安全还是比较重要的，虽然国内公司都不重视这个。数据安全不仅仅是研发侧打的日志是否有敏感内容、网络过程的数据安全，还包括研发同学捞取用户日志的申请流程等。
加密一般都是使用的对称加密，速度快一些。密钥都会存储在 app 内部，安全性高一些的还会通过图片来存储密钥。但总归是写死到本地的，破解人员可以静态资源或者二进制串改密钥等方式进行破解，还是有风险的。
这里可以借鉴 HTTPS，客户端生成公私钥对，把公钥给到服务端，让服务端把对称密钥通过公钥加密给到客户端。这样可以降低日志被解密的风险。
还有一个安全点就是基于 asl 和 oslog 的日志打印。很多同学不知道这个日志打印的内容，是可以通过 console.app 查看的，即使是线上包。很多大厂 app 都有这个问题，比如网络的重联、内存释放信息等调试数据，在大厂的 app 线上包，都有存在。如果打印了用户的一些敏感信息，就很危险了。

对于 PC 侧的用户动线等数据消费可视化，可以通过日志的 level、module、page、压后台、进前台等各种行为做可视化。

## FastImageCache&mmap

**mmap**在高性能日志场景都被使用了，包括微信的 xlog 和滴滴的 logan。主要用于解决 IO 频繁操作、脏数据写入、异常场景处理问题。
**FastImageCache**是对**mmap**用的比较深入的 iOS 图片缓存库。
它和 sdwebimage 等其他的图片缓存库相比，唯一的特色就是对解码后的数据进行本地缓存。
它本身的设计，是业务强依赖的设计，并不好直接使用。但是它的技术实现并不复杂，而且还有一些缓存周期等技术瑕疵。完全可以脱离它的设计，自己设计一套符合自身业务场景的图片缓存。

**FastImageCache**的核心就是下面一些代码：

```
// 通过 mmap 将文件直接映射到虚拟内存
_bytes = mmap(NULL, _length, (PROT_READ|PROT_WRITE), (MAP_FILE|MAP_SHARED), fileDescriptor, _fileOffset);

// 将上下文映射到虚拟内存，待上下文绘制完毕，生成的解码数据也就在虚拟内存里。通过 msync 将数据同步写入文件
CGContextRef context = CGBitmapContextCreate([entryData bytes], pixelSize.width, pixelSize.height, bitsPerComponent, _imageRowLength, colorSpace, bitmapInfo);
imageDrawingBlock(context, [_imageFormat imageSize]);
CGContextRelease(context);

int result = msync(pageAlignedAddress, bytesToFlush, MS_SYNC);

// 对虚拟内存进行读取，拿到文件中的图片解码 data，然后直接生成 uiimage 对象。
// 1. 通过 mmap 拿到文件缓存，不用通过内核态和用户态做二次拷贝
// 2. 通过解码数据直接生成 uiimage 对象，避免了 CPU 硬解耗时
CGDataProviderRef dataProvider = CGDataProviderCreateWithData((__bridge_retained void *)entryData, [entryData bytes], [entryData imageLength], _FICReleaseImageData);
CGImageRef imageRef = CGImageCreate(pixelSize.width, pixelSize.height, bitsPerComponent, bitsPerPixel, _imageRowLength, colorSpace, bitmapInfo, dataProvider, NULL, false, (CGColorRenderingIntent)0);
image = [[UIImage alloc] initWithCGImage:imageRef scale:_screenScale orientation:UIImageOrientationUp];
```

其他代码也有很多，但都是为了维持**FastImageCache**它自身设计的那份数据结构，包括 table/entry/chunk 等，就是不断的套娃，方便数据的检索。
**FastImageCache**最好不要直接拿来用，业务改动会非常大。自己模仿写一个不会太复杂，而且可以规避它那边现有的一些缺陷。
比如它有一个 maximumCount 参数用来设置每个 table 的最大 entry 容量，实际上内部没有太多的实现，想超多少超多少。而且这个值重新设置后，会因为配置内容发生改变，所有缓存文件会全部删除。
它内部对每个 mmap 的切片大小也有些小，也不支持外部改变。毕竟都是七八年前的开源库了，那时候存储和流量还都比较贵。
其他还有很多，比如缓存文件的种类、大小等设计，都有很大的局限性。
但是它里面的像素对齐和字节对齐是非常有用的，如果想写一套，这个一定要抄下作业。

___

又一年没看春晚，看样子是戒了，开心。
