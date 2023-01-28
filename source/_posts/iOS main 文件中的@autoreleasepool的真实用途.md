---
title: iOS main 文件中的@autoreleasepool的真实用途
date: 2020-05-05 16:20:56
categories:
- 技术
tags:
-  iOS
keywords: iOS,main文件,@autoreleasepool
---

这个问题，在很多年前其实是经历过血腥风雨的。

就是下面的代码，Xcode创建项目的时候，会生成下面的main.mm文件:

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

问题就是，main里面的autoreleasepool是做什么用的？

<!-- more -->
前几年是iOS炙手可热的疯狂年代，面试官们会经常问到这个问题，后来，这个问题后来越来越淡了，因为没有人能回答的好它。
甚至于，能回答这个问题的人，都是瞎扯淡，我可以这样确认。

这个问题，我曾经研究过很久，我把可能的牵强逻辑都灌进去，也无法得出结论。那时我给自己的答案是：
**我确认这个autoreleasepool没有任何用处，不管别人怎么说，除非拿出实锤，否则我确定它没任何用途。**
**苹果是一家特别的公司，不会恶意强奸用户，所以我相信它有特别的用处，但是我不知道是啥。**

显然，这是一个两面派的答案。一来我相信自己的逻辑判断，二来我相信苹果一定别有用途。

我本以为这个问题，至少有经验的iOS开发一定不会问，因为问就是自讨没趣，这是一个没有答案的问题。
如果有人能给出答案，那就像回答”鲁迅写的那个字妙在哪，体现了他当时什么心情“一样的天马行空，我可以确认能回答出来的答案绝对是天马行空的。

显然，我写这个文章的原因，是有人问了。
我甚至听到这个问题的时候，以为这个已经被淡忘好多年的经历过血腥风雨的问题又要卷土重来一遍。

我按捺不住自己的复杂心情，又打开了google，想看看最近是不是又有什么新的傻逼讲解出来了。
显然，我依旧一无所获。

那种不死心，很多时候是天生的，我在5月5号，写下文章标题，用来激励自己，这次，一定要找到答案。
中间断断续续的，把相关书籍的部分章节重点看看，把OC相关源码重点看看，和同事聊聊聊，一直到今天，5月9号。

我想看看Xcode创建Mac项目的main文件长啥样，如下：
```
// Mac app
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // Setup code that might create autoreleased objects goes here.
    }
    return NSApplicationMain(argc, argv);
}
```

反正我立刻就惊了，Mac和iOS的main文件竟然是不一样的。我立刻创建一个iOS项目，如下：

```
// iOS app
int main(int argc, char * argv[]) {
    NSString * appDelegateClassName;
    @autoreleasepool {
        // Setup code that might create autoreleased objects goes here.
        appDelegateClassName = NSStringFromClass([AppDelegate class]);
    }
    return UIApplicationMain(argc, argv, nil, appDelegateClassName);
}
```

我当场就笑了。

不知道Xcode的哪个版本开始就改变了项目的main初始代码，至少，我一直没有注意它。

这个能够引发血腥风雨和奇思淫计的问题，总算可以成为历史了。

现在如果有人问“main文件中的@autoreleasepool的用途”，就是光明正大的问自动释放池的释放原理了。
