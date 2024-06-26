---
title: 谈谈事件驱动
date: 2018-11-08 15:10:08
categories:
- 技术
tags:
- 计算机原理
- iOS
---

`事件驱动`这个技术方案，可以说实实在在影响了这些年编程界的技术方向。最实际的受用者应该就是`异步编程`了，如I/O。

我所认知到的语言，都是事件驱动的使用者，受益者，推动者。

<!-- more -->

很多朋友可能更多的停留在精通官方API阶段，还没有更深层次的认知计算机原理，不晓得代码是怎么工作的。我不精通官方API，计算机原理也认知浅薄，但在自己认知的语言范围内了解了一些。我感到庆幸，希望和大家分享。

### `事件驱动`的形象描述

A和家人去外婆家取了个号。A一直在门口等着，是阻塞。A出去玩一会，门分钟去外婆家门口看看到了自己号没有，是监听。A去一个衣服店看衣服去了，到了自己号的时候，收到外婆家发的短信消息后直接去外婆家，是回调。

上面这个例子，变种极多也千篇一律。串行并行、同步异步、阻塞非阻塞都可以用。我们用这个例子来说`事件驱动`。

这里分析第三种情况，A去看衣服去了，这个时候外婆家的号叫到自己，并短信通知了自己，这个行为的分析。

A查看短信的行为和外婆家发出短信的行为，就是我们分析的重点。

外婆家为了发出这个短信，需要耗时，甚至外婆家也不知道耗时多久。所以外婆家需要做的是，一定要在A的号到了的时候，准确及时的发出短信，这是一个事件的发出。

A虽然有手机，但是看不看短信是A的事情。所以A一定要在看到短信之后立即作出处理判断，这是一个事件的反馈。

但是A为了能够尽快吃到饭，多做了几手准备，他把西贝、海底捞几家店的号都拿了，打算谁先叫到自己，就去谁家。

所以A接受到的事件是多个并且不确定的。

`Life is thread`，我们把A比作一个线程。A能够及时响应各家店面发来的消息，原因就是线程里面有一个`while (true):{pass;}`这样的循环。依靠CPU这个超强大脑控制器，只要有事件需要通知到线程，线程里面的这个while循环就会获取到并及时处理。
    
所以`事件驱动`的本质是：**一方及时发出事件，通过CPU时间片实时轮转事件循环队列并告知到while循环以通知到另一方，另一方及时响应事件。**

### iOS中的使用
##### `Runloop`
`Runloop`可以说把`事件驱动`利用到了极致。你能想象，如果没有`Runloop`，你就真的不能使用iPhone手机了！
`Runloop`依托于线程。我们手势点击一个按钮，就是在操作主线程里的`Runloop`。
`Runloop`通过`事件驱动`在以下8种状态下实时循环切换，用于省电的同时又能够及时处理用户界面反馈。
```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```
##### 闭包
我们通过IOS里的`Block`闭包，可以在异步执行一串功能逻辑代码后接着处理闭包里的活。
```
- (void)someMethodThatTakesABlock:(returnType (^nullability)(parameterTypes))blockName;
```
```
[someObject someMethodThatTakesABlock:^returnType (parameters) {...}];
```
##### 还有其他非常多的使用，如通知等

### Python中的使用
Python近些年才完全开发完优秀的`协程`并开放使用。使得Python上`多并发`成为事实上的可能。这里的多，不是之前的几十几百，是上十万。
##### `Event_loop`+`协程`
Python里面使用多线程其实并不怎么爽，本身就是耗资源的语言，多线程切换更加雪上加霜了。通过协程，妥妥的解决了上十万的并发。
```
import asyncio
async def test(i):
	print("test_1",i)
	await asyncio.sleep(1)
	print("test_2",i)
loop=asyncio.get_event_loop()
tasks=[test(i) for i in range(5)]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```
##### `aiohttp`
网络访问上的多并发，对`协程`的进一步封装。

### Node.js中的使用
##### `Event_loop`事件循环
这个就没得说了，彻彻底底依靠`事件驱动`起家的语言。基于`javascript`和`V8`引擎起来的Node.js，就是完完全全的单线程语言。
可是要知道，Node.js就是以单线程中使用事件驱动处理高IO闻名世界的。
你可以想象一下，一个Node.js搭建的后台，每秒上千上万的并发，都是单线程在处理吗？
Java就是一个用户一个请求一个线程（又名`线程驱动`），服务器资源耗费真的大。
```
var fs = require("fs");
var debug = require('debug')('example1');

debug("begin");

fs.readFile('package.json','utf-8',function(err,data){
    if(err)  
        debug(err);
    else
        debug("get file content");
});

setTimeout(function(){
    debug("timeout2");
});

debug('end');

```
