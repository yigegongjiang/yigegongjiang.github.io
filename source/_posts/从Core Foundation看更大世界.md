---
title: 从Core Foundation看更大世界
date: 2020-05-23 16:51:05
categories:
- 技术
tags:
- 网络
keywords: 
---

![tPJlTK.png](https://s1.ax1x.com/2020/05/26/tPJlTK.png)

Core Foundation是被iOSer**忽略**的一个**重要**框架。说重要，因为Core Foundation提供了丰富的组件库，这些组件库可以很好的用于开发工作。
但之所以被忽略，因为很多开发工作，可以用更友好的Foundation框架替代。
Core Foundation有Foundation没有的功能，比如CFDictionary的Key元素无需实现NSCoping协议、CFArray可以不进行对象引用计数等。反过来，Foundation也有Core Foundation无法胜任的工作，最大的来说就是**自动引用计数**功能。
在iOS项目开发过程中，我们可以使用基于C语言的Core Foundation框架写一些业务功能逻辑，甚至有时候非用Core Foundation不可，因为它有Foundation没有的功能。

Foundation是用Objective-C语言写的，Core Foundation使用C和C++语言写的。我们都知道Objective-C是C的**超集**，所以认为Objective-C和C、C++混编是正常的。
那么，什么是超集？Objective-C是动态的面向对象的，C是静态的面向过程的，如何实现这个超集？
既然Objective-C可以和C、C++一起使用，那么Golang呢？我们可不可以用Go来做混合开发？

通过Core Foundation，可以有更大的认知空间。
比如各类高级语言在计算机中是如何运行的？Dart(flutter)可以做混合开发，原理是什么？Lua做热更新，它不是C语言也不是Objective-C语言，是怎么被计算机调用执行的？用Node.js写iOS代码，到底行不行？
各种风马牛不相及的高级编程语言，是否有各自的边界？

<!-- more -->

### iOS开发框架有哪些

框架是库的更高一层描述，这里的库一般指的是动态库，但说静态库，也完全没有问题。
比如，我们要写一个语音识别功能，我们写了1-n个动态库(a.framework、b.framework...)来完成这个功能。在项目最后，客户说只需要一个动态库，我们就把这1-n个动态库组合成1个动态库并命名GJAudioKit.framework，就可以叫GJAudioKit为语音识别框架了。
所以框架这个专有名字该怎么解释说明？
一来是众多库组合起来的意思。为了一个功能，需要写1-n个库。最后将1-n个库组合成1个库，这个库就叫一个框架。
二来框架是一个抽象，是比库更高层级的抽象。不管静态库还是动态库，都是目标文件(.o)的合集。而框架比库的抽象层级更高，表示一个功能、一个业务或者一个模块，比如语音识别框架。

iOS系统提供了很多框架给我们使用，详见下图：

![tiVp28.png](https://s1.ax1x.com/2020/05/26/tiVp28.png)

我们经常使用的框架都正在图中的分层结构中找到对应的影子。
也可以从Xcode的资源文件中查看，如下图：

![tiZ2kR.png](https://s1.ax1x.com/2020/05/26/tiZ2kR.png)

刚才说到框架有1-n个库组成，上图中的SwiftUI.framework框架里面只有一个SwiftUI.h头文件，而Foundation.framework框架里面有近130个头文件。我们可以理解为一个头文件即可单独生成一个动态库。

框架的理解就到这，值得一提的是，上面列出的框架，都是系统提供的，也都是C、C++、Objective-C写的。我们自己当然也可以开发需要的库或者框架，那么，我们必须要用C族语言开发吗？

### 如果用Go来写iOS框架会怎么样

手机和PC有很多不同，比如PC的硬件都是可以拆卸的，而手机一般都不会拆卸，所有硬件都是集成到一个主板焊死的，我们不能随便更换存储和内存。CPU也不一样，PC有很多种类的CPU支持，可以根据用户是美术生或者喜欢玩游戏而选择不同的产品型号，比如撕裂者等。而手机为了省电，都用的ARM架构CPU。
但不管手机和PC如何不同，有一个共同点是计算机发展几十年来不曾改变的，乃至手机和计算器都是一样的原理，那就是他们都依托发展于冯诺伊曼机。

程序需要执行，不能执行没有任何意义的程序，所以输入输出是必须的。而冯诺伊曼机的程序执行就是执行二进制。

所以从这点上看，高级语言再怎么变化，最终也跑不了CPU指令集二进制执行这个宿命。

PC上我们可以跑上千种语言，因为这些语言最终都是二进制。只要语言被编译汇编成能够被执行的指令集，那么这些语言就有被编写和执行的意义，不管是在PC上执行，或者手机或者树莓派上执行。

所以，Go当然可以在iOS手机上运行，不仅Go，Java、Ruby、Lua、Node.js都可以。那，怎么才能将Go写到iOS程序里面呢？
这就要分析C是如何被iOS系统执行的。因为他们都属于高级语言，如果C能够依靠一个逻辑被执行，那么Go按照这个逻辑也就可以执行。

#### C语言是如何被操作系统执行的

##### 系统调用和运行时

很久很久之前，是用纸带打洞进行编程。那时候CPU直接执行二进制。
现在就不行了，因为有操作系统存在了。操作系统是一个大管家，管理着所有的应用程序，通过合理的管理应用程序的内存，进行系统级别的控制。
操作系统该如何操作，才能管理应用程序呢？显然，控制了代码，就控制了所有。如果实际运行的代码，都在操作系统的管辖范围内，那么操作系统就想怎么控制就怎么控制了。
这个时候，我们就无法绕过操作系统直接让CPU执行我们的二进制了，而是需要让操作系统在中间做一个中间者，我们调用操作系统的接口，操作系统进而让CPU执行进入内核态(内核塌陷)，这个时候我们的代码才算被执行。这个操作系统的接口，就是**系统调用**。
操作系统提供了完善的服务，人们都装了主流的几大操作系统。因为我们的代码需要被用户拿去执行才有意义和价值，所以，我们的代码全都需要接受操作系统的控制。
所以现在，我们的C语言代码从开发阶段到被执行，是下面这样的：

![titA6H.png](https://s1.ax1x.com/2020/05/26/titA6H.png)

系统调用，都是汇编实现的，并实现了C的接口供用户调用。这里需要说明一点，几大操作系统都是用C语言提供的系统调用接口。不管用户态是什么类型的高级语言，系统调用提供的仅仅是C接口。

下面是部分系统调用接口，

![tiwf6U.png](https://s1.ax1x.com/2020/05/26/tiwf6U.png)

系统调用接口并不是很多，都是操作系统提供给外界的刚需接口，大约350个左右（不同系统的接口数量不同）。这个时候，C开发人员开发的时候都是调用open用来打开文件，调用brk来申请内存。显然和现实不太一样，我们开发的时候，都用的fopen打开文件，用malloc申请内存。这是为什么呢？

原因就是直接使用系统调用，非常困难。具体分两点来说：
1. 系统调用提供的接口都是基础接口，比较生硬且基础。程序员需要的一个很基础功能，可能需要调用好多个系统调用接口才能完成。
2. 系统调用是操作系统提供的。如果用户用Linux系统的系统调用接口开发了程序A，那么如果想让程序A在Windows系统上运行，那是不可能的，因为两个系统的系统调用接口完全不一样。

显然，C语言开发者直接进行系统调用，遇到了困难。而中间件可以解决所有困难，如果解决不了，那就再加一个中间件。

下面是添加了C Runtime Library（运行时）后的调用流程：

![tiyzB8.png](https://s1.ax1x.com/2020/05/26/tiyzB8.png)

运行时是一个中间层，用户写的代码，最终调用的都是运行时接口。这个接口可以专门为C语言提供非常丰富的接口调用，有下面四种情况：
1. 1个系统调用接口可以为n个运行时接口提供服务，比如malloc和free都使用了brk。
2. 1个运行时接口可以调用n个系统调用接口，比如w接口，需要x、y、z接口同时提供服务。
3. 1个运行时接口可以仅调用1个系统调用接口，这个时候是1-1关系。
4. 1个运行时接口可以不调用系统调用接口，如strcpy，专门的C语言字符串处理函数。

而且，不同系统提供的系统调用是不同的，但只要改变运行时，不需要修改用户代码，即可适配多平台。而每个系统都需要维护一套语言级别的运行时，这是必要且可行的。

这样，C运行时作为中间层，极大的提高了开发人员生产力。

##### 跨过运行时直接系统调用

这里有一点需要说明，C运行时虽然作为高级语言和系统调用的中间层，但也不一定非要过这个中间层不可。因为操作系统只管理系统调用，而上层如何调用系统调用，是不受约束的。如果有一门语言，完全没有运行时概念，用户代码直接对接系统调用完全没有问题，就是上面说的**系统调用生硬且基础**和**不能跨平台**两个缺点。所以C语言真实调用逻辑如下：

![tVJ2As.png](https://s1.ax1x.com/2020/05/28/tVJ2As.png)

开发人员在编写代码的时候，可以调用C运行时库接口，也可以直接进行系统调用。但还是那句话，这样用的人肯定不多，项目里面可能个别代码会如此实现，但一定是有足够把握才会这么做。

##### Windows API的存在

还有一点需要说明，Windows和Linux还有些不一样。Linux的CRT直接进行系统调用，而Windows又加了一层中间层，名叫Window API。这个中间层夹在系统调用和MCRT之间，如下图：

![ti7UyD.png](https://s1.ax1x.com/2020/05/26/ti7UyD.png)

Window API对微软意义重大，作为最出名对商业软件，Window API更好的保障了用户升级带来对兼容性，所以中间层真的很好用。

##### C运行时和C标准库的关系

下面额外补充一点，C语言规范中，出了标准，没有出实现。所以C语言的编译器和相关库版本非常多。
因为不同操作系统之间，有相同特性也有不同特性，所以不同操作系统的运行时接口有相同的也有不同的。
比如，内存分配，提供的api都是malloc。而windows有图形界面的绘图api，对应的linux就没有体现。
于是就把默认提供的api如malloc或者printf等叫做C 标准库。其他各自独有的也并入C运行时库。详见下图：

![tib9Ej.png](https://s1.ax1x.com/2020/05/26/tib9Ej.png)

C的运行时比较特别，主要因为C出了标准，却没有给实现，于是各家为政。所以C的运行时库里面包含了C标准库，还有其他接口如启动函数等。而对于其他高级语言，一般就没有运行时库包含标准库的概念，因为标准库和运行时库也是独立的。比如，Objective-C里面，系统提供了非常多的标准库即框架，这些框架都是动态库形式，但他们不是运行时，有单独的运行时库负责系统调用(OC比较特别，实际为对接C运行时而非系统调用，下面会详细说明)。

##### C语言运行总结

到这里，不知道大家有没有发现秘密，C语言能够被执行，有两个要点：
1. 系统调用。对外界提供统一的内核塌陷。
2. C运行时。提供C程序员开发的接口。

所以做为高级语言的C语言，能够在计算机和手机甚至嵌入式系统执行，核心就在于**系统调用**。我们编写的代码，在编译汇编链接后，都变成了对运行时的调用，而运行时对**系统调用Api**进行调用。系统调用由操作系统控制，所以操作系统才能严丝合缝的对我们编写的代码进行控制和管理。当然最终执行还是CPU执行指令，只是优先级、内存分配、线程调度等，都是操作系统控制了。

所以，只要一门高级语言，最终能够通过**L Runtime Library(语言运行时)**进行**系统调用**，那么，该语言就可以在操作系统的控制下，完成指令集的运行。

#### Objective-C是如何被操作系统执行的

上面我们了解了高级语言C语言运行的原理，那么我们紧接着看下Objective-C是如何被运行的。
我们都知道，Objective-C是C语言的超集，这个超集该如何理解呢？
C是面向过程的静态的，C不是动态化语言（不是完全动态化），因为函数调用在编译时候已经确定。
但是基于C语言的Objective-C语言，却是完完全全的动态化语言。Objective-C是面向对象的编译型语言，所有函数的执行都是在运行过程中确定，**超集**是如何做到这些功能的呢？

C的代码执行我们已经知道，我们写的代码，在编译后都变成C运行时函数调用的二进制。
这个C运行时作为中间层，干了一件大事，就是完成对系统调用的隔离和封装。
**超集**从字面可以理解，是在C的基础上做一些事情。
所以，我们可以想一下，如果在一个中间层上面再加一个中间层，在C运行时上面再加一个OC运行时，那么OC运行时是不是就可以做更多的事情？
比如，在Objective-C中，代码调用A类的a对象的method_1()函数，那么在运行时，我们希望调用method_2()函数。那么这么一个函数调用的变化，肯定不能依靠系统调用来做，它管内核状态，不管应用层事情。也不能C运行时来做，因为C运行时是C语言特有的功能，不会单独为高级语言Objective-C来做这个事，本身它也做不了，因为它是静态语言，自身都没有动态性。so，肯定有一层单独为Objective-C做了这个事情，这一层，就是OC运行时。

我们通过OC运行时源码分析一下对象创建的过程。
如果我们有一个OC_Person类，如下：
```
@interface OC_Person : NSObject
@property (nonatomic, copy) NSString *name;
@end
@implementation OC_Person
@end
```
现在，我们要创建一个对象person，即：
```
OC_Person *person = [[OC_Person alloc] init];
```
详细调用流程图如下：

![tVaMQ0.png](https://s1.ax1x.com/2020/05/28/tVaMQ0.png)

从上面的函数调用链，我们发现有两个关键点：
1. Objective-C进行对象内存初始化的时候，通过Objective-C的函数调用，最终调用到C语言的calloc()函数调用。
2. 内存分配的大小，是通过"->"结构体从一个struct拿到的。即调用一个结构体的instanceSize函数。（C++）

所以，我们在Objective-C里面创建一个对象并进行内存分配，开始的时候调用的是OC运行时，最终是调用了C运行时。我们创建的Objective-C对象本身，在运行时阶段，都是通过struct结构体获取值，所以对象在项目Build后，都转化成C/C++结构代码了。

我们在看下下面代码执行流程：
```
[person setName:@"x"];
```
详细调用过程如下：
`[person setName:@"x"]`->`objc_msgSend(person,SEL(setName),@"x")`->汇编代码在运行时阶段查找`struct OC_Person{...}`结构体中的setName函数的地址p_setName->`call Oxab435c2(p_setName)`。

上面函数调用过程分析如下：
1. Objective-C的函数调用，是通过汇编语言编写的objc_msgSend进行的中转。其实objc_msgSend本身就是一个中间层，是动态转发的入口，将函数调用中转到运行时阶段。
2. OC运行时阶段进行函数地址的查找，在找到对应的函数地址后，进行地址调用(函数执行)。

从上面对OC运行时的分析，我们可以看出，说Objective-C是C的超集，其实应该这样理解：
Objective-C是高级语言，在代码编译后，会调用OC运行时接口，进行相关操作如对象创建，方法查找等。而OC运行时接口的具体实现，则是依托C运行时实现的。
比如，我们创建的Objective-C对象，在编译后，都会转换成struct结构体的形式进行OC运行时调用。再比如，我们创建对象调用OC运行时的alloc接口，在内部，却是调用的C运行时的calloc接口(显然，calloc调用的是系统调用的brk接口)。
面向对象是面向开发人员的，OC运行时负责面向对象的Objective-C代码和C运行时之间的沟通。
Objective-C的标准库和C就不一样了。C的标准库上面说到，是C运行时的一部分。对于其他高级语言来说，标准库就是单纯为应用层封装的动态库，不属于运行时的一部分了。

下面是具体的流程图：

![tVhrvT.png](https://s1.ax1x.com/2020/05/28/tVhrvT.png)

我在之前文章中，也有一个Objective-C和运行时库的说明：[Objective-C 和 Runtime](https://www.yigegongjiang.com/2020/04/08/Objective-C%20%E5%92%8C%20Runtime/)。

#### Go该如何被iOS系统执行

我们已经分析了C和Objective-C在iOS操作系统上运行的原理。
我们可以确定，我们写的代码，只要最终能够对接系统调用并编译成二进制交由操作系统运行，那么我们的代码就能运行。
我们写的代码都是高级语言代码，比如我们写的高级语言的函数调用：`[OC_Person alloc]`，这个函数在编译后我们假设为`call Oxa1b2c3`，其中`Oxa1b2c3`是OC运行时的`_objc_rootAlloc`函数的虚拟地址。
在`_objc_rootAlloc`内部会调用`calloc()`进行C运行时函数调用，我们假设为`call Oxd4e5f6`。到此为止，我们自己写的[OC_person alloc]代码，我们知道在代码区里面，那么`Oxa1b2c3`和`Oxd4e5f6`这两个运行时的函数在哪里呢？
运行时说到底，也是代码。运行时有两种存在形式，一个是动态库，一个是静态库。
我们的操作系统都默认有C的动态库运行时。所以我们在Linux A电脑编译出来的ELF执行文件，在Linux B电脑上是可以直接运行的，就是因为C运行时库是用动态库的形式在执行文件启动后进行链接的。不仅仅操作系统，只要是计算机，大差不差都有C的动态库运行时，所以C语言才如此通用。因为只要写了C语言，不出意外到哪里都可以跑起来，除非用了特定系统的api。
那如果有一个操作系统，真的没有C的动态库运行时，是不是就不能支持可执行文件了呢？
运行时存在两种形式，一种动态库，还有一种静态库。我们在编译可执行文件的时候，可以把运行时打到可执行文件中，这样刚才说到的`Oxa1b2c3`和`Oxd4e5f6`运行时函数就打到可执行文件中了，即代码区。这样，及时操作系统没有安装C的动态库运行时，可执行文件一样可以跑起来。只不过，这样的化，可执行文件就会变大，因为包含了静态库运行时的大小。
所以Objective-C能够在iOS和Mac上运行，就是因为这两个系统里面，有动态库OC运行时。
那Objective-C能不能在安卓手机或者树莓派上面运行呢？因为Objective-C不支持运行时的静态库链接，而安卓和树莓派上没有动态库OC运行时，所以就运行不了Objective-C App了，因为找不到对应的函数调用，即上面说到的`Oxa1b2c3`和`Oxd4e5f6`。

##### Go静态库运行时的必要性

所以，Golang该如何在iOS系统上执行？Golang本身是高级语言，肯定有运行时库，分别有动态库和静态库两个版本。因为iOS操作系统本身没有Golang运行时，那么在编写Golang的代码后，在编译链接的时候，把Golang的静态库链接到最终的执行文件中(静态库或者动态库，或者叫框架)，那么这串Golang编写的代码，就能够在iOS系统上完美的运行起来。
这个时候，Golang运行时需要做那些事情呢？
1. Golang需要做一个静态库运行时，链接到执行文件中。因为iOS系统本身只有Objective-C运行时、C运行时、C++运行时，没有Golang运行时。
2. Golang原本肯定没有考虑运行在iOS上，所以Golang的运行时对接了Windows和Linux的系统调用。那么现在，Golang的静态库运行时就需要对接iOS的系统调用。

当上面两个步骤完成，我们就可以通过Golang编写代码并导出framework库，在iOS系统上被执行。
我们写出Golang库，肯定还是希望被Objective-C调用，因为Objective-C和C支持混编，而Go有一个库`CGo`，可以让Go和C连通，所以Objective-C这个时候就可以放心的调用Golang开发的库/框架了。

下面是Golang在iOS系统上的运行流程图：

![tVqdIS.png](https://s1.ax1x.com/2020/05/28/tVqdIS.png)

幸运的是，Go开发iOS所需要的库/框架，目前已经有发行版了，即[Go Mobile](https://github.com/golang/mobile)。我测试一下，简单一行代码，在编译后的嵌入动态库中，也有1.5M，原因就是这个动态库中，有Go的静态运行时和CGO静态库。

##### 其他高级语言如何编写iOS需要的动态库

其实不止Go，Node.js也一样可以用来开发iOS的动态库，有这个[Node.js for Mobile Apps](https://github.com/JaneaSystems/nodejs-mobile)。
Go和Node.js能够写iOS动态库，那么按照同样的逻辑，其他高级语言也一样可以做这件事，比如，游戏开发中，很多就用Lua来做热更新，Lua代码要被执行，也需要一个Lua的静态运行时嵌入到库中。

上面说到的，还都是业务功能库，不是UI。那Flutter就是完全依靠Dart语言来做跨平台的混合开发方案。Flutter的库会让iOS App的包体积增加15-25M，就是因为里面有一系列的运行时和相关UI组件库存在。
我们通过上面Go的大小可以发现，运行时库本身没有多大，Flutter库比较大的原因，是Flutter通过Dart完全重新实现了自己一套UI框架，所以代码量肯定是巨大的，框架体积自然就增大了。

##### 限制高级语言的枷锁是什么

所以我们可以发现，至少在iOS上面，是没有高级语言限制的，只要高级语言有这个运行在iOS系统上的需求，都能实现。
而Go如果后期希望像Flutter一样实现UI框架，也一样没有问题。限制它们的，仅仅是业务需求罢了（实际上，对于Go和Node.js的iOS动态库开发，需求不大，所以都是试探性发展，因为C和C++已经足够优秀，用Objective-C来开发本身也够用了）。举例而言，C++是编写稳定后台服务的热门语言，而基于C++的Qt，就可以用来做跨平台的GUI。而Swift初期被用来开发iOS/Mac App，现在也一样可以用作服务器开发。甚至Javascript只是浏览器端的脚步语言，引入V8引擎后，JS已经花开两朵，前端和Node.js后台发展的都非常棒。

我们也可以认知到，高级语言的存在，只是特定场景的需求。如果当年苹果不开发Objective-C，用Java来开发iOS App，也完全可以的，只是苹果需要一套自己的能够被私有控制的开发体系。
语言是用来完成特定场景的工作任务，如果用Objective-C来写服务器的I/O多并发，显然没有Go和Node.js的事件驱动来的吞吐量大。而Objective-C后期能不能实现协程、多进程等特性？当然可以，就是需要不需要而已。
**限制语言功能及发展的，仅仅是它的业务场景，而不在于语言本身或者操作系统**。

### Core Foundation 和 Foundation 的区别

我们在上面已经研究了语言和框架。框架和库的关系，在文章开头也已经说明。
这里就来研究一下Core Foundation和Foundation两个框架的区别和联系。

Core Foundation是基于C开发的，Foundation是基于Objective-C开发的。但是有一点，Foundation是基于Core Foundation封装并实现了Core Foundation所没有的部分。
我们可以用下图来表示Core Foundation和Foundation的关系：

![tZCYEd.png](https://s1.ax1x.com/2020/05/28/tZCYEd.png)

Foundation用Objective-C封装了Core Foundation的C组件，并实现了额外了组件供开发人员使用。而Core Foundation也有一些Foundation没能彻底封装的功能，这些功能是Core Foundation特有的。

下面可以看一下Foundation和Core Foundation的组件库都有哪些：

![tZPSqe.png](https://s1.ax1x.com/2020/05/28/tZPSqe.png)
![tZCzrD.png](https://s1.ax1x.com/2020/05/28/tZCzrD.png)

从图中，我们可以看到，Foundation的组件是多于Core Foundation的，比如NSBundle在Core Foundation就没有体现。而NSArray就和Core Foundation的CFArray是对应的。反过来，Core Foundation的CFTree和CFBitVector在Foundation里也没有体现，或许是在其他组件中使用到了这两个算法库。

因为Core Foundation是C实现的，虽然Objective-C能够兼容并调用C，但是和C相互通信并转换，就不那么容易了。
其实Objective-C和C直接通信就像Go和C直接通信一样，是高级语言之间的通信。Go有CGO库完成了这个中间层，Objective-C虽然基于C，有得天独厚的优势，但是如果没有官方实现，那还是会出现高级语言之间的代沟。
举例来说，现在有下面两个C和Objective-C代码：
```
C:

typedef struct person{
	int age;
	char *name;
} Person;

---

Objective-C:

@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSUInteger age;
@end
@implementation Person
@end
```
我们现在创建一个C语言的p变量和Objective-C的pp对象，尝试将他们互通，代码如下：
```
C:
Person p = (Person *)malloc(sizeof(Person));
p->age = 10;
p->name = "GJ";

Objective-C:
Person pp = Person.new;
pp.age = 100;
pp.name = @"GJ2";

开始通信1：
pp = <Conver>p;
NSLog(@"pp name is %@", pp.name);

开始通信2：
p = <Conver>pp;
printf("p name is %s\n", p->name);
```
C是面向过程的，Objective-C是面向对象的。上面的p和pp的格式转化，目前来看的确是没有办法完成的。也就是说，缺少`<Conver>`这个环节。
Objective-C本身是可以直接使用C代码的，虽然转化比较困难，但可以在不转化的前提下，直接调用C结构体变量进行使用。但是C却没有办法直接调用Objective-C对象了，所以这个时候可以写一个转换层，来完成这个工作：
```
Objective-C to C:

C_Person * conver(Person *p) {
    C_Person *c_p = (C_Person *)malloc(sizeof(C_Person));
    c_p->age = (int)p.age;
    c_p->name = [p.name cStringUsingEncoding:NSUTF8StringEncoding];
    return c_p;
}

---

使用：

p = conver(pp);
printf("p name is %s\n", p->name);
free(p);
```
我们可以看到，通过这样中转的方式，我们可以将C和Objective-C相互转换并通信。

显然，大家也发现有些费事。虽然这些转换如果真的要写C代码，那么就必不可少。但是如果使用Core Foundation，那会方便很多。
我们刚才说过，Foundation是封装的Core Foundation，苹果开发了一个强大的功能，即**桥接（Bridge）**。通过桥接，可以非常方便的实现C和Objective-C的数据转换，比如下面：
```
CFMutableDictionaryRef cf_mu_dic = CFDictionaryCreateMutable(kCFAllocatorDefault, 10, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
NSMutableDictionary *oc_dic = (__bridge_transfer NSMutableDictionary *)(cf_mu_dic);
// CFRelease(cf_mu_dic)；// 注意，这里因为__bridge_transfer缘故，就可以不用执行CFRelease释放内存了。
```
对于Core Foundation使用过程中产生的变量，都可以通过桥接的方式，变成Foundation对象。桥接帮我们做了格式转换的同时，也帮我们做了ARC。
刚才，我们在执行`p = conver(pp)`的时候，大家注意，后面使用了C语言的内存释放，即`free(p)`。Core Foundation本身也有引用计数，但是没有自动计数即ARC。所以Core Foundation的对象释放的时候，需要调用CFRelease，那么在桥接到Foundation后，就可以使用Objective-C的ARC了，非常方便。
桥接中的`__bridge/__bridge_transfer/__bridge_retain`可以很方便的帮我们做对象管理转移操作，我们就不需要手动去释放内存了。

这里还是要补充一点，基于C语言的Core Foundation之所以能作为Objective-C开发框架，就是上面提到的，只要是高级语言，只要有相关运行时，就可以用来开发组件/库/框架。

### Core Foundation 和 C 与 Objective-C 的转换

#### 桥接（Bridge）

我们从上面Core Foundation和Foundation之间了解到，通过桥接，可以很好的转换Core Foundation和Foundation对象。
桥接做了两件事，一个是**自动引用计数**，一个是**格式转换**。

我们先说一下格式转换，因为桥接的转换局限性很大。
我们上面把C的`struct person`结构变量转换成Objective-C的`Class Person`，需要自己写类似于`C_Person * conver(Person *p)`的转换函数。说明C和Objective-C之间转换本身是不能直接进行的。
但是Core Foundation的CFArray、CFString等和Foundation的NSArray和NSString等转换，通过桥接就可以直接转换。这是因为Core Foundation比较特别。Core Foundation是苹果自己写的C代码，所以在桥接的时候，苹果拥有Core Foundation的数据结构和Foundation的对象细节，所以桥接可以自动完成转换工作。
而我们自己写的C结构体变量，和Objective-C对象之间，就不能很好转换了，如果我们写了C代码需要和Objective-C进行转换，就必须自己写一个中间层了。
这就是桥接对于数据格式转换的局限性，准确来说，桥接对数据格式转换，的确只在Core Foundation里面才有体现，毕竟如上所说，苹果自己知道Core Foundation和Foundation之间的所有细节。
这对于我们来说，其实已经完全够用了，因为我们真实业务开发场景，如果需要避免Objective-C的运行时带来的消耗，的确可以通过Core Foundation来编写代码。

下面再说桥接的另一个大杀器，那就是自动引用计数。
Core Foundation在和Foundation进行转换的时候，可以通过`__bridge/__bridge_transfer/__bridge_retain`进行自动引用计数控制，这个不在细说。
这里介绍C和Objective-C之间通过桥接进行引用计数控制。引用计数是针对Objective-C对象来说的，我们看一下Objective-C对象和C之间的转换：
```
    Person *oc_p1 = Person.new;
    oc_p1.name = @"GJ";
    oc_p1.age = 10;
    
    // 1. 假设这里因为业务代码需要，我们将OC对象转成C指针进行传递
    void *c_p = (__bridge void *)(oc_p1);// c_p = 0x0000600003977500
    // 2. 程序执行过程中，有各种原因可能会导致oc_p1对象计数-1，比如离开块区域等。这里我们通过置nil进行模拟
    oc_p1 = nil;
    // 3. 这里需要把之前由C指针存储的指针还原成Objective-C对象
    __weak Person *oc_p2 = (__bridge Person *)(c_p);// 这里有"__weak"避免计数影响。因为默认是"__strong"，计数会+1
    // 4. 这里使用还原后的Objective-C对象
    NSLog(@"%@", oc_p2.name);
```
上面我们使用**__bridge**进行Objective-C和C的强制指针转换，表示不对计数进行任何改变。
在代码执行到第3步的时候，就会奔溃。
因为第1步前，堆对象的计数为1，第1步没有改变计数，堆对象计数还是1。
经过第2步，堆对象不在有引用计数了，所以堆对象就被释放了。
在第3步，想要使用C的c_p指针的时候，这个指针所存储的`0x0000600003977500`堆地址，已经变成野指针了，使用的时候直接会崩溃。

下面看下**__bridge_retained**和**__bridge_transfer**的作用：
```
    Person *oc_p1 = Person.new;
    oc_p1.name = @"GJ";
    oc_p1.age = 10;
    
    // 1. 假设这里因为业务代码需要，我们将OC对象转成C指针进行传递
    void *c_p = (__bridge_retained void *)(oc_p1);// c_p = 0x0000600003977500
    // 2. 程序执行过程中，有各种原因可能会导致oc_p1对象计数-1，比如离开块区域等。这里我们通过置nil进行模拟
    oc_p1 = nil;
    // 3. 这里需要把之前由C指针存储的指针还原成Objective-C对象
    __weak Person *oc_p2 = (__bridge_transfer Person *)(c_p);// 这里有"__weak"避免计数影响。因为默认是"__strong"，计数会+1
    // 4. 这里使用还原后的Objective-C对象
    NSLog(@"%@", oc_p2.name);
    // 5. 这里继续将C指针存储的指针还原成Objective-C对象
    __weak Person *oc_p3 = (__bridge_transfer Person *)(c_p);
```
这里代码运行情况分析如下：
第1步之前，堆对象计数为1。经过第1步后，**__bridge_retained**会使得计数+1，堆对象计数变成2。
经过第2步，堆对象计数变成了1。
第3步**__bridge_transfer**会使得计数-1，堆对象计数变成0，堆对象被释放。
第4步会打印null，因为oc_p2本身为null。
第5步，程序崩溃，因为c_p指针存储的堆对象已经释放，指针此时为野指针。

从上面两个例子，我们可以看到，在和C进行赋值的过程中，桥接帮我们做了引用计数的工作。和Core Foundation的转换过程中的计数规则是一样的。
我们在赋值过程中，使用**__bridge_retained**和**__bridge_transfer**可以有效的降低崩溃风险，因为这两种bridge方式，帮我们做了引用计数的加和减。
单独进行**__bridge**赋值的时候，引用计数没有改变，相当于同一时间，有多个指针指向堆对象，但是对象的计数却和指向指针的个数不一致。如果对象被释放，很可能还有指针在指向，这个时候使用就会发生野指针。
通过**retained**和**transfer**，在赋值过程中，加1和减1是同步的，这样可以有效降低对象计数和指向指针个数不一致的野指针风险。

#### 通过桥接给C和Objective-C赋值的风险

通过上面两个例子，或者写更多其他Objective-C和C指针赋值的代码后，就会发现这样写代码的风险非常大。最大的风险就是野指针和内存不释放。
如果完全写Objective-C的代码，OC运行时已经帮我们处理了引用计数和对象释放后指针自动变nil问题，所以我们大概率不会出现野指针和内存不释放情况（**OC运行时的Weak表帮我们处理了对象释放后指针自动变nil。而Objective-C的引用计数的内存管理方式，也容易因为循环引用导致内存不释放，这是引用计数管理内存的天然缺陷**）。
但是在C赋值嵌入进来后，即使通过桥接进行计数管理，也依旧摆脱不了随时崩溃的风险。原因就是因为对象被释放导致野指针随时可能会发生，或者对象无法释放导致内存泄漏。
对于经常写C代码的程序员来说，应该不会担心这些问题，因为他们已经习惯内存需要手动管理。被拥有自动内存释放机制娇生惯养的程序员们，就需要注意这个风险了。
比如下面代码，就很发生内存泄漏：
```
void func {
    int number = 10;
    char *c_chars = (char *)malloc(sizeof(char) * number);
    memset(c_chars, 0, number);
    int i = 0;
    for (; i < number - 1; ++i) {
        *(c_chars + i) = 'a' + i;
    }
    
    NSString *oc_str = [NSString stringWithCString:c_chars encoding:NSUTF8StringEncoding];
    free(c_chars);
    NSLog(@"%@", oc_str);// 打印“abcdefghi”
}
```
上面代码中，前面用C语言申请了10个字节的堆空间，然后开始赋值被转成Objective-C的NSString对象。
oc_str是ARC控制的，出了func函数作用域，内存就会被释放。可是如果忘记写`free(c_chars);`这行代码，就会导致10字节的内存泄漏。像这样的内存细节，防不胜防的同时又会慢慢耗尽内存空间。

所以，**如果需要避免Objective-C的运行时带来的消耗而想采用C写业务，最好使用Core Foundation，它和Foundation之间的桥接非常完美，一般不会出问题。而自己写C进行混用，野指针和内存不释放是挥之不去的地雷**。

### Core Foundation的使用

Core Foundation只是一个非常优秀的框架，但是苹果用C写的Core Foundation框架和Objective-C写的Foundation框架，不是iOS框架的全部。框架是库的抽象，用Golang等其他高级语言，一样可以写出优秀的框架。Dart就是举足轻重的例子。

上文的截图中，给出了Core Foundation框架里面都有哪些好用的组件，比如CFString、CFDate等。下面的一些示例，是用Foundation不好实现的。

#### CFRunloop介绍

iOS的Runloop水还是很深的。我也写了Runloop的一篇文章，一直在草稿中未能发布。因为牵涉面太广，如事件驱动、线程休眠、自动释放池、UI刷新等。通过Runloop能够更加清楚明白的理解App运行的原理，也可以做非常多有用的东西，如主线程卡顿监控、线程保活等。
Foundation提供了NSRunloop供我们开发人员使用，但是NSRunloop有一个大坑，对于不了解Runloop的开发人员来说，很容易陷进去。
网上有很多Runloop的介绍，在介绍让线程执行一段时间的时候，会使用`[[NSRunLoop currentRunLoop]run]`。我揣摩本意，发现他们并不是想要让线程永久长活，但是却使用了`run`函数。这样会使得当前线程永远无法释放，是永远。因为NSRunloop里面run函数是对`CFRunLoopRun()`函数的true循环封装，当结束一次循环后，NSRunloop会立刻再次调用`CFRunLoopRun()`函数，没有任何办法可以销毁当前线程的Runloop。这样，项目里面就永远的多出来一条可能已经不再需要的线程。主线程就使用的这个逻辑。
在CFRunnloop里面，仅有两种方式安全启动线程的runloop，分别为`CFRunLoopRun()`和`CFRunLoopRunInMode`，其中`CFRunloopRun`还是语法糖。这两种启动方式，都是一次循环，客户端可以自行控制啥时取消Runloop，有效的降低Runloop未知风险。相关源码如下：
```
CFRunloop.c

// RunLoop 运行循环
void CFRunLoopRun(void) {    /* DOES CALLOUT */
    int32_t result;
    do {
        // 调用RunLoop执行函数（默认运行default Mode）
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}

// 切换并运行到对应的mode（运行modeName参数对应的mode）
SInt32 CFRunLoopRunInMode(CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) {     /* DOES CALLOUT */
    CHECK_FOR_FORK();
    return CFRunLoopRunSpecific(CFRunLoopGetCurrent(), modeName, seconds, returnAfterSourceHandled);
}
```
iOS 的runloop，就是通过调用`CFRunLoopRunSpecific()`->`__CFRunLoopRun()`实现的。其中，NSRunloop的run函数，相当于下面代码：
```
- (void)run {
    do {
        CFRunloopRun();
    } while(true);
}
```
所以，线程永远也无法销毁。因为CFRunloopRun()函数会在Mode切换或者手动调用`CFRunLoopStop()`等情况下执行完毕，但是外部的do-while true循环，永远结束不掉。

这里，如果需要写Runloop相关的代码，我强烈建议使用CFRunloop，而不要使用NSRunloop。相比来说，CFRunloop提供了比NSRunloop更加细致化的Api，相比之下，NSRunloop就寥寥无几了。

下面是我写的一些CFRunloop测试代码，因为Core Foundation是C语言写的，所以里面的组件都是面向过程的调用方式，和面向对象有些不同：
```
    self.thread = [[NSThread alloc] initWithTarget:self selector:@selector(createThread) object:nil];
    [self.thread setName:@"test thread 1"];
    [self.thread start];

- (void)createThread {

    NSLog(@"the thread is [%@]", [NSThread currentThread]);

//    NSRunLoop *runloop = [NSRunLoop currentRunLoop];
//    self.port = NSMachPort.new;
//    self.port.delegate = self;
//    [runloop addPort:self.port forMode:NSRunLoopCommonModes];
//    [self addObserver];
        
    
    CFRunLoopAddCommonMode(CFRunLoopGetCurrent(), (__bridge CFStringRef)@"dadada");
    
    CFRunLoopPerformBlock(CFRunLoopGetCurrent(), kCFRunLoopCommonModes, ^{
        printf("abc");
    });
    
    NSLog(@"the runloop is [%@]", [NSRunLoop currentRunLoop]);

    [[NSRunLoop currentRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:10]];
//    CFRunLoopRunInMode(kCFRunLoopDefaultMode, 5, NO);
    NSLog(@"---end");
}
```
```
主线程下调用"addObserver"，可以实时查看主线程的Runloop状态

- (void)addObserver {
    CFRunLoopObserverRef runloopObserver = CFRunLoopObserverCreateWithHandler(
        kCFAllocatorDefault, kCFRunLoopAllActivities, YES, 0,
        ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
        
        switch (activity) {
            case kCFRunLoopEntry:
                NSLog(@"--1 即将进入loop kCFRunLoopEntry--");
                break;
            case kCFRunLoopBeforeTimers:
                NSLog(@"--2 即将处理timer kCFRunLoopBeforeTimers--");
                break;
            case kCFRunLoopBeforeSources:
                NSLog(@"--3 即将处理source kCFRunLoopBeforeSources--");
                break;
            case kCFRunLoopBeforeWaiting:
                NSLog(@"--4 即将休眠 kCFRunLoopBeforeWaiting--");
                break;
            case kCFRunLoopAfterWaiting:
                NSLog(@"--5 即将从休眠唤醒 kCFRunLoopAfterWaiting--");
                break;
            case kCFRunLoopExit:
                NSLog(@"--6 即将退出loop kCFRunLoopExit--");
                break;
            default:
                break;
        }
    });

    CFRunLoopAddObserver(CFRunLoopGetCurrent(), runloopObserver, kCFRunLoopCommonModes);
}
```

#### CFDictionary介绍

Foundation里面有NSDictionary与之对应，如果我们希望用我们自定义的对象为key，存储与NS字典中，直接存储是不行的：
```
// 定义Person类

@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSUInteger age;
@end
@implementation Person
- (NSString *)description {
    return [NSString stringWithFormat:@"name:%@, age:%lu", self.name, (unsigned long)self.age];
}
- (void)dealloc {
    printf("Person Dealloc.\n");
}
@end

- (void)func {
    Person *p = Person.new;
    
    NSMutableDictionary *oc_dic = NSMutableDictionary.new;
    [oc_dic setObject:@"" forKey:p];// 这里会崩溃
}
```

上面代码中，如果我们把自定义Person类的对象p作为key存储到NSMutableDictionary中，运行时是会崩溃的。
因为Foundation规定，字典的key必须要实现`NSCoping`协议，字典在添加属性的时候，是调用[key copy]作为字典key的。
改写如下：
```
@interface Person : NSObject<NSCopying>
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSUInteger age;
@end
@implementation Person
- (id)copyWithZone:(nullable NSZone *)zone {
    Person *p = Person.new;
    p.name = self.name;
    p.age = self.age;
    return p;
}
@end
```
如果我们使用Core Foundation，就可以避开这个限制，即Person类不需要实现`NSCoping`协议，如下：
```
    Person *p = Person.new;
    
    CFMutableDictionaryRef cf_mu_dic = CFDictionaryCreateMutable(kCFAllocatorDefault, 10, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
    CFStringRef cf_str_value = CFSTR("value");
    CFDictionaryAddValue(cf_mu_dic, (__bridge const void *)(p), cf_str_value);
```
CFDictionary默认会对Key和Value做retain，所以我们使用**__bridge**即可。当p被当作key加入cf_mu_dic后，p的引用计数已经变成2了。
如果我们使用**__bridge_retained**，如下：
```
    Person *p = Person.new;
    
    CFMutableDictionaryRef cf_mu_dic = CFDictionaryCreateMutable(kCFAllocatorDefault, 10, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
    CFStringRef cf_str_value = CFSTR("value");
    CFDictionaryAddValue(cf_mu_dic, (__bridge_retained const void *)(p), cf_str_value);

    p = nil;
    CFDictionaryRemoveAllValues(cf_mu_dic);
    // 这里因为"__bridge_retained"缘故，p置空和移除CFDictionary所有元素后，对象的引用计数还是1，所以内存泄漏。
```
**这里因为"__bridge_retained"缘故，p置空和移除CFDictionary所有元素后，对象的引用计数还是1，所以内存泄漏**。

CFDictionary的key不需要实现`NSCoping`协议这一特性，YYModel就有使用，这也是YYModel使用CFDictionary的最终原因：
```
YYMemoryCache.m

    _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
    CFDictionarySetValue(_dic, (__bridge const void *)(node->_key), (__bridge const void *)(node));
    CFDictionaryRemoveValue(_dic, (__bridge const void *)(node->_key));
    CFDictionaryGetCount(_dic);
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));

    CFRelease(_dic);
```
YYModel使用CFDictionary，就是因为缓存对象是各式各样的，极大可能都是没有实现`NSCoping`协议的。
因为YYModel通过一个`__unsafe_unretained`类型的双向链表来保存对象，所以YYModel需要一个容器来持有缓存对象防止被提前释放。
为了加快查询对象的速度，使用查找复杂度为1的hash map结构即字典(CFDictionary)，而非数组(CFArray)。

CFDictionary还有一个巨大特性，是可以吊打NSDictionary的，那就是可以自行控制引用计数。下图表示CFDictionary的创建函数及相关函数调用：

![te7jRH.png](https://s1.ax1x.com/2020/05/28/te7jRH.png)

我们举例一下，
```
_dic = CFDictionaryCreate(CFAllocatorGetDefault(), keys, values, n, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
```
这里创建一个_dic变量，这里每个key都会经过`kCFTypeDictionaryKeyCallBacks`结构体获取到`retain/release/copyDescription/equal/hash`进行函数调用。
比如，**如果两个key一样，那么equal就会比对出true，第二个key元素就会被过滤**。注意，这里和NSDictionary不一样，NSDictionary是完全hash map table，两个元素如果一样，就会通过**拉链法**或者**开放寻址法**进行存储。但是在CFDictionary里面，如果两个元素equal为true，则会过滤另一个。
然后，一个key被存储的时候，会调用retain函数进行引用计数+1。这里调用的是系统默认的，如图中所示，如果我们用自己的retain函数代替系统的，就可以实现引用计数的多变性：
```
void * Custom_CFDictionaryRetainCallBack(CFAllocatorRef allocator, const void *value) {
        return value;// 系统默认为return CFRetain(value);
}
```
我们通过改写一个key retain 函数，就可以改变CFDictionary的key在retain时候的计数是否+1。
如果执行`CFDictionaryRemoveAllValues(cf_mu_dic);`，则字典中所有元素都会被移除，这个时候每个key都会被调用release函数执行引用计数-1操作，我们也可以重写：
```
void Custom_CFDictionaryReleaseCallBack(CFAllocatorRef allocator, const void *value) {
    // 系统默认为CFRelease(value)，现在啥都不做
}
```
我们改写key release函数，就可以使得key被移除释放的时候，引用计数不在-1。

这里，我们的操作性非常强，我们可以提供自己的函数地址，就可以实现多样化的CFDictionary引用计数逻辑，详细代码如下：
```
const void * custom_dictionary_key_retain(CFAllocatorRef allocator, const void *value) {
    return value;
}

void custom_dictionary_key_release(CFAllocatorRef allocator, const void *value) {
}

const void * custom_dictionary_value_retain(CFAllocatorRef allocator, const void *value) {
    return value;
}

void custom_dictionary_value_release(CFAllocatorRef allocator, const void *value) {
}

- (void)func {

    CFDictionaryKeyCallBacks custom_dictionary_key_call_backs = {
        0,
        custom_dictionary_key_retain,
        custom_dictionary_key_release,
        CFCopyDescription,
        CFEqual,
        CFHash,
    };
    
    CFDictionaryValueCallBacks custom_dictionary_value_call_backs = {
        0,
        custom_dictionary_value_retain,
        custom_dictionary_value_release,
        CFCopyDescription,
        CFEqual,
    };

    Person *p = Person.new;
    
    CFMutableDictionaryRef cf_mu_dic = CFDictionaryCreateMutable(kCFAllocatorDefault, 10, custom_dictionary_key_call_backs, custom_dictionary_value_call_backs);
    // 这里的cf_mu_dic对于key和value的引用计数完全改变了，key和value在加入和移除的时候，引用计数都不会被CFDictionary改变
    CFDictionaryAddValue(cf_mu_dic, (__bridge const void *)(p), (__bridge const void *)(p));
}
```

这里，我们描述了很多CFDictionary相比于NSDictionary的不同，有如下：
1. CFDictionary的key不需要实现NSCoping协议，NSDictionary的key如果没有实现NSCoping协议，则会运行时崩溃。YYModel等开源库主要就是使用了这个特性。
2. CFDictionary的key如果相等，在元素不会被插入。NSDictionary则会通过**拉链法**和**开放寻址法**进行数据存储。
3. CFDictionary的key和value的引用计数，都可以自行控制。NSDictionary的key用的[key copy]，value用的retain。

所以，CFDictionary相比NSDictionary来说，扩展性也更强。

___

最近一直在喝Luckin Coffee，最近因为收入造假，快要被纳斯达克下市了。