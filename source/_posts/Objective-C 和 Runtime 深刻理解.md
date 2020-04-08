---
title: Objective-C 和 Runtime 深刻理解
date: 2020-04-08 18:43:31 08:00
categories:
- 技术
tags:
- C
- iOS
keywords: Runtime,Objective-C Runtime, OC Runtime, 结构体, 指针, 运行时, 运行时库
---

**运行时(Runtime)**本身是一个非常普通的概念，每个编程语言都会有运行时，非iOS Objective-C特有。
概念上理解，**运行时，就是程序执行的过程**，每个编程语言，只有运行后执行特定的任务才有价值，所以每个编程语言，都有运行时。
而Objective-C的Runtime显然不仅仅是程序执行过程这么简单，它一举将基于**面向过程的语言C**而实现的**面向对象的语言Objective-C**变成了**动态语言**。
为什么Objective-C的Runtime有些难理解，因为运行时从概念上理解非常简单，但是Java、Python、C它们的运行时都是不一样的，运行时期间可以做很多事情，所以运行时理解起来还是比较抽象和高阶。
如果把Runtime改名为**OCDR**（Objective-C Dynamic Resolution，即**Objective-C动态决议**），剥离运行时和运行时库的概念，那么很多人都会轻松掌握Objective-C Runtime的核心。而Objective-C Runtime本质上的确就在践行**动态决议**这么一个过程。

<!-- more -->

Objective-C Runtime使得函数调用变成了动态消息传递，并且可以在执行过程中对Object进行增删改查，所以更准确的说，Runtime使得Objective-C变成了动态语言。

#### 动态语言分析

首先理解一下各类语言的一个区分分界，那就是**动态语言**和**静态语言**的区分（非动态类型语言和静态类型语言）。

##### 静态语言

静态语言比较容易理解，我们通过高级语言写好的代码，经过预处理、编译、汇编、链接后，就形成了机器码（二进制文件）。这个时候，函数名、变量名都保存在符号表中，并拥有对应的虚拟地址。而我们调用一个函数，就是通过机器码调用的，汇编示例如：'call 0X12345678'，这里call代表函数调用，0X12345678代表函数的虚拟地址。
这个机器码执行的过程中，PC指令寄存器会不断的记录下一行将要执行的指令，然后不停的执行下去(延伸一个知识点，如果下一行指令不再内存中，操作系统会发现页缺失，然后从硬盘中将对应缺失页的指令拷贝到对应的真实内存中，然后继续执行)。
这里我们可以发现，我们通过高级语言写好的函数，在编译后，就已经不可改变了。程序执行后，CPU内部的运算单元、控制单元、数据单元，就有条不紊的按照机器码执行就好了。
那么如果我们想改变一个函数的实现呢？比如本身调用的A函数，运行期间想调用B函数，有办法实现吗？对于静态语言来说，不行！
那么如果我们想改变一个对象的结构体呢？对于静态语言来说，不行！
所以静态语言，对于函数的定义，在编译期就必须是明确的，如果找不到函数的实现，就会编译错误。
举例来说，如C语言。

##### 动态语言

动态语言相比静态语言，也比较容易理解了，在运行时代码可以根据某些条件改变自身结构，如函数的调用，自定义类的生成和对象的创建等。那这样的语言就是动态语言。
举例来说，如Objective-C语言。

##### 分析

C是静态的，Objective-C是动态的，而Objective-C是基于C实现的。那Objective-C是如何基于C实现动态特性的呢？

C有它的运行时，不过程序执行过程中，CPU完全掌控机器码执行流程，所以C的运行时不能多样化，基本上就是按照程序员写的代码执行顺序执行。这也是C快的原因，因为CPU直接执行，不用考虑那么多复杂的情况。

那Python如何实现动态的呢？Python也有它的运行时，不过说来巧妙，它不是编译性语言，不会像C一样打包成机器码直接执行。Python是解释性语言，代码执行到哪一行，就即时编译该行形成机器码执行，所以Python非常容易实现动态，只要代码运行过程中适当的添加if语句对将要执行的对象进行替换，就能很好的实现动态，鸭子Duck模型也就来源于此。

那Objective-C呢？Objective-C和C一样是编译性语言，项目打包后会经过编译链接等处理变成机器码。
Objective-C实现动态就要运行时了，因为在运行时做了很多操作，以至于很多人叫它“运行时系统”。
之所以前面说Objective-C的Runtime应该改名为"Objective-C Dynamic Resolution，Objective-C动态决议"，就是因为Objective-C的Runtime主要实现的两个点都更加符合“动态决议”这个命名：
1. 通过objc_msgsend(a_object,a_object_method,xx)这个中间者函数，动态解析被调用的函数。
2. 通过指针、数组的指针、链表指针、全局数据，间接操作函数列表、属性列表等，动态的对Object进行增删改查。

#### Objective-C Runtime 函数调用动态性

之所以很多人说Objective-C的函数调用并非函数调用，而是消息传递，其原因就是Objective-C的函数调用很不一样。
在C中，我们调用一个函数，会这样写：
```
Node *head = NULL;
...生成链表...
reverseLinkList(&head);// 反转链表
```
这里，reverseLinkList函数的调用，在汇编里面为：call 0x12345678，其中0x12345678就是reverseLinkList这个符号名的虚拟地址。
这里我们可以发现，reverseLinkList函数的虚拟地址在编译的时候就已经定下来了，后期无法做到动态性。
再来看一下Objective-C的实现，因为Objective-C是面向对象的语言，所以我们调用对象的一个方法：
```
Person p = Person.new;
[p realAge];
```
这里，[p realAge]这个函数的调用，p有自己的虚拟地址，realAge函数也有自己的虚拟地址，但是在汇编之前，该函数已经被编译成了这样：
objc_msgsend(p, realAge)。
如此，Objective-C通过objc_msgsend这个完全汇编写成的中间函数，在函数内部通过**运行时库**对p进行superclass和isa的访问，乃至最后实现三次动态函数转发操作。
所以Objective-C实现消息传递，就是依靠objc_msgsend这个中间函数来实现，如果objc_msgsend的虚拟地址为0x87654321，那么[p realAge]函数被执行的时候，实际汇编为：call 0x87654321。

上面说完了objc_msgsend这个中间函数完成动态解析的操作，那它是如何进行superclass和isa的访问，又是如何访问属性和方法列表的呢？

#### Objective-C面向对象的本质：C的struct（结构体）

通过Runtime源码，我们可以发现，Objective-C里面一切皆对象，乃至UILabel、NSTimer等。而这些对象是以什么样的数据结构存在呢？
都是struct（结构体）!我们的成员变量，成员属性，方法等，都保存在结构体里面。
所以这里我们可以深刻理解一下，为什么说Objective-C是C的超集，**因为面向对象的Objective-C语言完全就是依靠面向过程的C语言发展起来的，乃至于我们写的面向对象的代码，最后都会变成面向过程的执行流程**。
我们可以查看苹果公开的runtime.h文件，都是面向过程的函数调用。如：

> objc_setAssObjective-CiatedObject(id _Nonnull object, const void * _Nonnull key, id _Nullable value, objc_AssObjective-CiationPolicy policy)

你看，这完全就是面向过程的编程。

因为struct是我们对象的存储结构，所以我们可以通过*class_getProperty(Class _Nullable cls, const char * _Nonnull name)*等库函数直接对struct内部保存的数据进行访问。
比如我们要访问a_object的a_object_method方法，那么就要通过a_object_struct，根据superclass找到类A_struct，然后遍历A_struct里面的methods数组，找到a_object_method方法并执行。

struct结构体在编译后就已经确定，那如何做到动态性如增加方法呢？struct的实现是C和C++混编的，但大体逻辑不变，都是通过指针实现的。
比如说，struct里面有一个方法的数组指针，那么我们动态添加方法的时候，只要通过这个数组指针就可以添加到对应的方法数组中，这样就完全不会改变struct的结构(甚至如果数组需要扩容，也和struct没有任何关系)。

因为struct在编译后就已经确定，所以能做动态性的方案不外乎这几点：
1. **通过数组指针，如方法列表等**
2. **通过链表指针，如NSNotification等**
3. **通过全局对象，如关联对象等**

所以指针才是C和Objective-C最终执行的终极形态，不断通过指针间接的对数据区进行增删改查。
而**Objective-C的Runtime就是不断通过指针对struct结构体进行增删改查数据处理来实现动态特性**。

#### Objective-C Runtime 总结

Objective-C Runtime用的最多的是objc_msgsend，所有的函数调用都会通过这个消息传递完成。
编译成机器码的函数调用，本身无法实现动态性，通过objc_msgsend这个中间层巧妙的实现了。
而objc_msgsend的汇编执行又得反过来操作运行时库，又回到了对象的struct结构体上面。
**Objective-C Runtime本质就是通过指针操作struct的增删改查来实现Objective-C的动态特性**。

C语言中一切皆指针，基于C发展起来的Objective-C，也是一样。
而指针也是中间层思想的体现。本身无法直接操作的数据，通过指针这个中间层，间接的对数据进行操作。
