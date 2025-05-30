---
title: Shell和进程
date: 2022-10-04 11:29:29
categories:
- 技术
tags:
- 计算机原理
- Shell
- C
- 内存
---

曾经有位老师问过：在 Linux 的 Shell 中运行程序时，操作系统是怎样对程序进行处理的吗？
我当时回复是这样的：操作系统对 Shell 的执行，是靠 Shell 解释器完成的。在操作系统运行后，Shell 解释器本身就加载并运行了。其中如 pwd,cd 这些是内部命令，本质是函数调用，可以直接使用。ls 这些是外部命令，需要 fork 一个新进程执行当前命令。一个 Shell 脚本，有很多个这些内外部命令组成，通过 Shell 解释器逐行解释完毕后执行。Shell 解释器也是一个应用程序，本质是一个 C 程序，不过在该程序中，手动模拟了函数调用栈，和 JVM 有相似之处。所以 Shell 解释器，也有静态库/动态库/静态链接/动态链接这些，为 Shell 命令的执行保障护航。
现在看起来，当时的回复虽然没有大的问题，但还是不够。只是浅表的认知了 Shell 解释器的用途，但是对 Shell 并没有深刻的理解。
最近看到子 Shell(SubShell)，发现 Shell 和进程之间的关系非常密切。可以从进程的角度来理解**操作系统是怎么运行 Shell 的**。
这篇文章的目的，是让你只关心 Shell 脚本的语法，而其他细节，都变成顺理成章。其实各个高级语言也都是这样，语言本身其实并不难，像工具一样使用而已。

<!-- more -->

## 0x00 温故知新

对于 Shell 的运行，有一些前置知识需要粗略的描述一下。这些前置知识非常重要但也可以不写(属于基础范畴)，因为它对理解 Shell 非常重要，所以这里还是加以补充说明。
没办法，计算机的知识都是一层套一层垒起来的，一个已经定型的知识点会不断的被重复使用，如果不能直接使用了，就再加一个中间层去使用它。
如果已经对这些内容比较熟悉，还是建议快速的过一下，也许会有新的发现。

### 解释型语言 VS 编译型语言

目前的高级语言，都是需要被转化成指令码才能被计算机运行。而因为转化的时机不同，高级语言被分为两个阵营：**解释型**和**编译型**。
其中对于解释型语言，为了更好的效率，也更多的加入了编译型的影子，就是 JIT 即时编译。所以也可以认为还有一个阵营，即**解释编译混合型**。

#### 编译型

像 C、C++、Objective-C、Swift、Go 这些，都是编译型的。典型的特征就是：项目代码需要被处理成可执行文件后，才能运行。

* 如果改动了哪怕一个字母，也需要把编译、汇编、链接这套流程走一遍，才能重新生成可执行文件并运行。有时候项目比较大，重新编译运行一下需要耗时 5+ 分钟，**所以开发效率很低**。
* 可以用 C 写一套编程代码，但针对 Mac、Linux、iOS、Android 这些平台，是需要针对不同平台，用不同的编译方式打不同的可执行文件后，才能运行。这就是编译型语言的另一个特点，就是**不能跨平台**。
* 编译型语言的最大优点是**超高效率**，编译型语言是经过前后端编译后形成的二进制可执行文件，CPU 的运行指令直接存储在二进制可执行文件中。编译器还会对指令做进一步优化，优化幅度非常大，以使得可执行文件运行的更快。
这里说的高效率，仅仅是说 CPU 可以直接从内存中对可执行文件进行取指、译码、运行，这一套流程非常快。而不是说语言本身高效率，因为不同语言侧重对方向不一样，有些 CPU 密集型，有些 IO 密集型，从这个维度来说高效则更侧重业务本身。

#### 解释型

像 Python、Java、Javascript、Shell 都属于解释型。典型的特征就是：项目代码无序被处理成可执行文件，就可以执行。解释型语言和编译型的特征都是相反的。
编译型需要经过前后端编译后，生成可执行文件。而解释型只需要经过前端编译，即词法分析、语法分析、语义分析三个步骤。如果精简一下，语义分析还可以不要，靠开发人员维护语义的正确性。
通过词法分析将代码转化成 N 个 **Token**，通过语法分析将 **Token** 转化成**抽象语法树(AST)**，而后通过**解释器**深度遍历 AST 即可将项目代码执行完成。

下面先针对最简化版本的解释型语言运行过程进行说明。
解释型语言的**开发效率非常高**，因为可以做到逐行翻译代码，所以修改了 N 行代码，就把修改的部分重新词法、语法分析一下，整合到 AST 中去。(编译型语言也非常渴望这个能力，有一种 hot reload 技术实现方案)。

解释型语言有一个核心，就是解释器。每个语言都会有不同公司做不同的解释器，刚才提到的解释型语言，就有Phthon 解释器(PVM)、JVM 虚拟机、V8 引擎、bash 解释器、zsh 解释器等等。
解释器承载了一个很大的技术地盘，就是**跨平台**。同样一套 js 编程代码，只要有解释器，就能够运行且等同运行环境一致。因为解释器抹平了端的差异，使得**一致的代码按照一致的运行逻辑被解释器执行**。
而解释器本身一般都是用 C 写的二进制可执行文件，所以解释型语言的运行，离不开编译型语言做基础。这一点很重要，解释型的代码之所以能够被逐行执行，是编译型语言先打包成可执行文件，当文件执行后，不断的逐行读取解释型代码并解释执行。而解释器的二进制可执行文件本身，也会有代码区和函数，这些函数也可以执行并且和解释型代码逐行翻译执行是不同的，这些函数就是 Shell 里面的**内部命令**，比如 pwd 命令，其实就是执行解释器本身的一个 pwd 函数。Shell 的**外部命令**就比如 ls，在解释器内部并没有一个 ls 函数与之对应，ls 实际上是另一个可执行文件，后面说到 Shell 内部 & 外部命令的时候再详细说明。

开发效率高和跨平台都是解释型语言的优点，而缺点也就是编译型语言的优点，就是**低效率**。
最简单的例子，对于 **1 + 2\*3** 这个加乘运算，在编译型可执行文件里面就是几个寄存器调用(几个时钟周期)的事情，但是在解释型逐行翻译的过程中，需要先被翻译成 5 个 Token 并生成 AST 树，这里的每一个 Token 还有读取操作和加法&乘法两个运算，至少需要 7 个函数的调用。而每个函数的内部还有一些额外函数调用，整体下来没有 20 个函数调用是下不来的。这和编译型语言来说速度上就是天壤之别。

#### 解释编译混合型

上面最简化版本的解释型语言描述，其实就是解释器初期版本，是纯粹的解释型语言执行流程。这个时候应用在执行效率没有那么高但是重视开发效率和跨平台要求的场景。后来，解释器做了极大的升级，主攻执行效率问题，引出了虚拟机的概念。
虚拟机对于执行效率问题的解决方案核心在两个方面，一个是通过**栈机/寄存器机**来模拟编译型语言的函数调用栈，一个是增加 JIT 实时编译的能力使得代码在运行时被编译成二进制执行，此时执行效率等同于编译型语言。
从这个角度来看，虚拟机可以称作解释器的升级版本，或者把虚拟机叫做解释器也没有问题。但是把这个中间层叫做虚拟机这么一个大的名字，显然能力不限于此。
实际上虚拟机在设计之初，就是模拟了一个物理机器，有自己的指令集和内存管理，还有并发/IO/线程调度能力，就是一个虚拟化的硬件+操作系统。从这个角度来看，虚拟机已经远远超出解释器范围了，毕竟最纯粹的解释器，就是对 AST 进行一次深度遍历。

虚拟机做的第一步，就是**不再进行 AST 深度遍历**。
代码执行的宏观表现，就是函数的调用。大多语言都是从 main 函数开始，不停的在子函数里面执行下去。所以无规则深度遍历可以切换成子函数的调用维度，通过对每个子函数的优化来提升整个代码的执行效率。
在编译型语言里面，函数栈帧是通过 CPU 执行可执行文件的指令在内存栈区维护的函数调用栈。虚拟机就需要在代码层面模拟这套栈帧。这里就需要定义一套栈和栈帧的数据结构，在虚拟机这里做入栈和出栈操作。
对函数调用栈有更深了解的，可以查阅：[从汇编角度理解 “函数调用栈” 和 “有栈协程”](https://www.yigegongjiang.com/2023/stackForFunc/)

虚拟机做的第二步，是将**复杂的逻辑前置**。
AST 这棵树还是非常复杂，如果依旧在这个树上做深度遍历，执行效率难以有质的提升，不做深度遍历用其他的方式遍历也没有质的提升。概括就是**底层数据结构不良好，上层算法难以用上力**。
这里虚拟机自己定义了一套私有指令集，比如 **iadd** 指令，用来做加法运算。这套私有指令是和 CPU 的指令集完全不一样的，因为解析这套私有指令的是虚拟机本身，而不是 CPU。
有了这套私有指令集后，虚拟机只要认这套符合自己规则的私有指令集并执行就可以了，再也不用管 AST 这棵树了。
这里中间层又会发挥作用了。首先 AST 上面说了是语法分析的产物，这一个环节必不可少。而虚拟机需要的又是私有指令集代码，不在关心 AST 了。所以这里需要有个工具把 AST 转化成私有指令集。
这个工具就是**字节码编译**。虚拟机把自己的私有指令集定义为 8 位长度，即一个字节，最多有 255 个指令。因为每个指令都是一个字节，所以虚拟机把它叫做字节码。而**字节码编译**要做的事情，就是把 AST 转化成字节码。

虚拟机做的第三步，是**指令优化**。
AST 这棵抽象语法树结构现在已经变成了字节码指令结构。对于 **1+10** 这样的运算，数据结构上就不再是 AST 树，而是 **load x, y** 这样的指令了。
是指令就需要操作数据，对于数据的存取，虚拟机有两种不同的实现，分别是**栈机**和**寄存器机**。
栈机是通过栈这个数据结构来对数据进行存取。对于 *1+10* 这个运算，转化成的字节码是这样子：**iconst_1;bipush 10;iadd;**，首先将数字 1 放入栈，然后将数字 10 放入栈，最后做加法运算就是把栈连续 pop 两次，把两次拿到的 1 和 10 做加法运算，运算结果再 push 到栈里面供其他消费。
寄存器机对于数据的存取主要依靠寄存器。对于 *1+10* 这个运算，转化成字节码是这样子：**Ldasmi 1; Star r0; Ldasmi 10; Star r1; Add r0 r1; Star r2;**，首先将数字 1 读取写入寄存器 r0，然后将数字 10 读取写入寄存器 r1。最后将 r0 和 r1 寄存器的值做加法后再写入寄存器 r2 供其他消费。
显而易见，**寄存器机是要比栈机快很多的**。从 AST 转换成寄存器机要比转换成栈机困难很多，主要是虚拟机的寄存器也是映射的 CPU 的寄存器，数据是有限的。如何有效的处理这几十个寄存器显得非常重要了，而栈机依靠栈这种数据结构，只要内存够用，栈机的数据读取就不是问题。
目前，Java 虚拟机 JVM 使用的栈机，V8 引擎、PVM 使用的寄存器机。

虚拟机做的第四步，是**JIT 即时编译**。
虚拟机发现某块代码经常被执行，那么虚拟机就会做一件事情，把这块代码编译成 CPU 指令集的二进制。然后把编译好的二进制放到内存的一块区域，并调用系统调用接口赋予这块内存可执行权限。
后续再次调用这块代码的时候，就直接转到内存处直接执行编译后的 CPU 指令。这里就和编译型的执行效率一样高了，都是直接跑在 CPU 上。
到这一步，解释型其实已经过渡到编译型了，只是编译的时机不同。编译型是必须做了后端编译，打成可执行文件后才能运行。**JIT 是某块代码经常被运行后再做后端编译，并把产物放到具有可执行权限的内存区域后运行**。
从这个角度来看，就不能再说 Java 是解释型语言，所以不够高效了。很多人抨击 Java 是解释型、和编译型相比中间有个 JVM 耗时层等等，最终指向 Java 慢或者效率不高。这就是对技术这个产业认知不到位。**技术这个产业是只要有利益，需求都能满足，只要利益够大，需求就会被超出预期的解决**。更多的摇摆因素是时间周期不固定，但需求一定会被解决。Java 在有了 JIT 之后，就是满负荷的编译型语言，哪还有什么效率差？即使说 GCC 或者 LLVM 耕耘多年后端能力底子深厚，Java 这些年的 JIT 能力相比也不会有太多出入了。至于 Python 的 PVM 虚拟机也有 JIT 能力但效率还是不如其他语言，那还有语言本身的限制在，比如 Python 的GIL 锁、Python 动态语言特性等。

### 两种进程创建方式 By fork & exec

在 Linux 系统下，进程创建只有一个方案，就是 fork 系统调用。
如果要创建一个新的进程，那么就得找到一个已经存在的进程，在这个进程里面调用 fork api，然后生成一个一模一样的新进程（可以认为一模一样，肯定会有差异）。新进程和已经存在的进程是父子关系。
从这个角度来看，整个 Linux 操作系统的进程图谱就是一个多叉树。操作系统启动的时候会创建 1 号进程 init。后续所有的进程的祖先，都是这个 1 号 init 进程。
对于 fork 有两个返回值，理解起来其实并不难：

```
int cal() {
  // 这里是非常复杂的计算，耗时严重
  int i = xxx;
  return i;
}

int main() {

  int index = 0;
  pid_t pid;
  pid = fork();
  if (pid == 0) {// 子进程
    printf("This is sub process, and pid is: %d\n", getpid());
    
    printf("the index is: %d, address is: %p\n", index, index);
    index = 10;
    printf("the index is: %d, address is: %p\n", index, index);
    
    int ret = cal();
    // 和父进程通过进程间通信将 ret 值给到父进程
  } else {// 父进程
    printf("This is origin process, and sub process pid is: %d\n", pid);
    
    printf("the index is: %d, address is: %p\n", index, index);
  }

  return 0;
}
```

上面 C 语言中，调用 fork 系统调用后，会有两次返回。为了保障子进程的优先级，一般子进程会先返回。
第二次返回的时候，pid 是子进程的进程号。这个时候代表父进程的执行流程。父进程后面该怎么做就怎么做，不受影响。
但是第一次返回的时候，pid 是0，表示当前处于子进程。进程是应用级别的单位，即两个应用肯定是两个进程。而 fork 的作用就是复制一个一模一样的进程出来，所以这个时候，表示新的进程的代码区也执行到了这里。可以这么理解，你已经使用 Telegram 15 分钟，在使用期间，你和 N 个好友聊天过，并且查看了 M 次订阅和 T 次群聊。但是在 15 分钟这一刻，Telegram 调用了 fork 接口，那么就打开了一个新的 Telegram 进程，在这个新的 Telegram 进程里面，你同样和 N 个好友有一样的聊天，也看了 M 次同样的订阅和 T 次同样的群聊，以至于新的 Telegram 进程里面，局部变量、全局变量都是一样的。
所以，fork 的这一刻，是两个进程的分界岭，之前的代码逻辑全部一样，在这一刻的数据也都是一致的。但之后就各走各的路了，不然开新进程干嘛呢。
我们可以在新的进程里面调用一个复杂的运算，比如上面的 cal 函数。在子进程完成运算后，将运算结果通过进程间通信给到父进程，这样可以有效使用多核 CPU 了。
**所以 fork 为什么有两次返回？其实根本不是 fork 函数有两次返回，没有一个函数能够返回两次，这违背了函数调用栈的原理**。详见：[从汇编角度理解 “函数调用栈” 和 “有栈协程”](https://www.yigegongjiang.com/2023/stackForFunc/)
**而是这本来就是两个完全一样的进程，执行到了同一个内存代码地址。在这之前，其中一个进程偷懒，复制了另一个进程的执行流程。在这之后，两个进程就各走各的路了**。

这里为什么父子进程拥有完全一样的局部变量、全局变量以及堆栈，是因为从进程的数据结构层面做的复制，所以虚拟指针这些都是一样的，映射的物理内存现在也还是一样的。
之后就各走各的路了，是因为进程数据结构复制的时候做了标记，后面两个不同的进程对物理内存进程写操作的时候，会把虚拟内存通过 MMU 单元映射到不同的物理内存上。比如代码区，两个进程肯定要执行不同的代码了，那么前面做的标记会出现缺页异常，从磁盘拿到不同的可执行文件的指令放置到新的内存代码区中。
这个技术叫做**写时复制**，可以有效提高子进程的创建效率。
这里也会出现一个现象，**就是同样的虚拟地址，对应的数据确实不一样的**。比如上面的 C 代码中，子进程将 index 赋值为 10，但是在父进程中读取的 index 还是 0。但是这个时候两个进程的 index 的虚拟地址都是一样的。**其实就是同一个虚拟地址，通过两个进程的 MMU 映射到了不同的物理内存栈区地址。他们已经是两个进程了，相互隔离，互不影响了**。
对于虚拟地址和 MMU 不理解的，可以看深度说明：[内存分段与分页](https://www.yigegongjiang.com/2022/memory/)

#### exec 

前面开头就说到，在 Linux 系统下，进程创建只有一个方案，就是 fork 系统调用。
但是 fork 产生的新进程，是和原来进程一模一样的。这对于有些进程来说，或许并不希望这样的结果。
进程是应用程序的基本单位，我们打开 Twitter 和 Telegram，其实就是打开两个进程。那对于 Twitter 和 Telegram 来说，其实他们希望他们有一个纯净的虚拟环境，而不是从父进程带进来很多杂七杂八的全局变量啥的。
这和 fork 的机制有些矛盾，所以这个时候就需要 exec 了。也叫做**创建进程的第二种方式，即连续调用 fork() 和 exec() 两个 api**。
exec 的作用很清晰，清空当前进程的所有数据，包括变量、代码区，完全变成一个新的进程环境。
比如上面 C 代码：

```
int main() {

  int index = 0;
  pid_t pid;
  pid = fork();
  if (pid == 0) {// 子进程
    exec();
    printf("the index is: %d, address is: %p\n", index, index);
  } else {// 父进程
    // other
  }
  return 0;
}
```

在子进程中调用 exec 后，就找不到 index 局部变量了，会运行时报错。因为这个时候数据都被清空了，代码区也不在了，所有都是新的。
如果像这样：**exec load twitter;**(伪代码)，就可以唤醒 Twitter 进程，Twitter 就可以从 0 加载起来了。（这里不说 Twitter 从 main 函数开始执行了，因为 main 函数不是一个程序第一个执行的函数，前置还有动态库加载等很多操作，详细可参考《程序员的自我修养》）

其次，exec 是一个接口簇，比如 execl() 等等。不同的接口有不同的表现形式。

最后，还有一个重要的点，并不一定非要调用 fork 了才能调用 exec。在任意时刻都可以调用 exec。只是调用 exec 后，当前所在进程的数据都会被清空。如果 exec 之后没有执行单元或者指定单元执行完毕，当前所在进程也就会被销毁了。

## 0x01 Shell 和 Shell 解释器

Shell 是我们对于命令行指令的统称。有时候我们也直接把 bash 叫做 Shell，虽然这有些不对。
我们可以这样定义 Shell，就是**用于和内核进行交互的用户应用层软件**。
在硬件层，是 CPU、内存这些硬件设备，我们虽然可以直接使用，但是那会很痛苦，于是操作系统出现了。操作系统可以帮我们做非常多的事情，比如进程管理、时间片轮转等。
操作系统是一个大而全的东西，它有一个核心，即**内核**。整个操作系统以及上层应用，都会通过内核和硬件层通信。所以内核对上层应用做了收口，我们经常说的系统调用，也就是调用的内核开放出来的 Api。
Shell 就是在内核的上一层，通过对内核进行各种系统调用，来完成各个命令的功能。而使用这些命令的人，即用户。所以 Shell 是一个应用层的应用程序。

即然 Shell 是一个应用程序，那 Shell 就是我们打开的终端软件吗(比如 iTerm)？也不是。
我们使用的终端，早期叫控制台。那时候一台电脑只有一个输入输出，就是通过控制台来操作。
后来电脑支持了多用户，一个控制台不够用了，就每个用户一个虚拟终端用来接入。这个虚拟终端也叫做终端模拟器(终端仿真器)。
**所以 iTerm 这些，都是终端模拟仿真器，简称终端**。

其实 Shell 是对一个系列的应用程序的统称。这些系列包括 bash 解释器、sh 解释器、zsh 解释器等。
bash、sh、zsh 这些，我们称为脚本语言，也是上面说到的解释型语言。对于这些语言，还不需要使用虚拟机用来提速，所以他们使用**解释器**对脚本指令逐行翻译执行即可。
脚本文件是没有提前编译，就在终端里面被执行了，这就是 bash 解释器做的事情。而 bash 解释器逐行翻译脚本的过程，就是上面的**解释型语言**里面说到的 AST 深度遍历。

## 0x02 Shell 内部命令 & 外部命令 

bash 解释器本身是一个应用程序，通过 C 语言编译成的。它本身是完整的 COFF 格式的可执行文件，是编译后的产物。
即然是可执行文件，那么就有代码区及函数。而我们使用的命令也分为内部命令和外部命令，比如 pwd 就是内部命令。
pwd 其实就是代码区的一个 pwd() 函数实现。内部命令是在 bash 解释器应用程序内部完成的，就像 main 函数可以对应一个 main 命令一样(假设)。

除了内部命令，bash 解释器还可以执行 ls 这样的外部命令。
外部命令的执行流程和内部命令是完全不一样的。外部命令本身都是一个应用程序，比如 ls。它的执行会是这个样子：

```
pid_t pid;
pid = fork();
if (pid == 0) {
  exec ls;
} else {
  wait(NULL);// 等待子进程执行完成
}
```

就是在当前进程创建子进程后，就 wait 等待子进程执行完成。而子进程就通过 exec 初始化进程环境然后执行另一个应用程序。
大家回想一下终端里面操作命令的时候，是不是就和 wait 这种情况一样？我们执行 ls 后，就不能输入了，等 ls 执行完毕后，我们才能继续输入命令。

从上面来看，**内部命令，其实是解释器内部的一个函数实现。而外部命令，其实是另一个应用程序**。
理解了前面说的**解释型语言**，Shell 命令其实就这样。

## 0x03 Shell 和 SubShell

Shell 里面有一个很重要的概念，是子进程和子 shell。其实子进程就是 fork + exec，子 shell 就是 fork。
那子进程和子 shell 有什么不同呢？看这里：

```
~ ❯ name=wanger                                                                    02:45:08

~ ❯ echo "current process - the name is ${name}."                                  02:49:32
current process - the name is wanger.

~ ❯ { echo "current process - the name is ${name}." }                              02:49:53
current process - the name is wanger.

~ ❯ (echo "fork process - the name is ${name}.")                                   02:51:03
fork process - the name is wanger.

~ ❯ cat d.txt                                                                      02:51:44
#!/bin/zsh
echo "fork&exec process - the name is ${name}."
~ ❯ bash d.txt                                                                     02:51:48
fork&exec process - the name is .
```

对于全局变量 name，在当前进程和 fork 进程均能正确读取，但是在子 shell(fork&exec) 里面就读取不到了。
这个现象，就是前面**两种进程创建方式 By fork & exec** 里面说到的，前面的理解了，这里就搬一下场景。
对于 *bash x.sh 和 ./x.sh* 执行方式，默认都是使用的 *fork + exec* 进程创建方式。当然也包括 ls 这些外部命令。
对于小括号组合命令、命令替换、管道，默认都是使用的 *fork* 进程创建方式。
对于大括号组合命令，则不创建进程，在当前进程执行。

验证当前是子进程还是 subShell，有两个环境变量：SHLVL(子进程) 和 BASH_SUBSHELL(子 shell)(zsh 使用 ZSH_SUBSHELL)。打印如下：

```
~ ❯ echo "SHLVL-${SHLVL}, ZSH_SUBSHELL-${ZSH_SUBSHELL}"                            03:00:34
SHLVL-1, ZSH_SUBSHELL-0

~ ❯ { echo "SHLVL-${SHLVL}, ZSH_SUBSHELL-${ZSH_SUBSHELL}" }                        03:06:16
SHLVL-1, ZSH_SUBSHELL-0

~ ❯ (echo "SHLVL-${SHLVL}, ZSH_SUBSHELL-${ZSH_SUBSHELL}")                          03:06:28
SHLVL-1, ZSH_SUBSHELL-1

~ ❯ cat d.txt                                                                      03:07:12
#!/bin/zsh
echo "SHLVL-${SHLVL}, ZSH_SUBSHELL-${ZSH_SUBSHELL}"
~ ❯ zsh d.txt                                                                      03:07:23
SHLVL-2, ZSH_SUBSHELL-0
```

因为打开终端后，当前 Shell 环境就是一个子进程，所以 SHLVL 默认是 1。从上面可以看到，子 shell 场景 ZSH_SUBSHELL 会 +1，在进程场景 SHLVL 会 +1。

## 0x04 Shell 变量作用域（全局变量 & 环境变量）

对于 Shell 的变量作用域，从上面的 name 全局变量和 fork&exec 的分析，其实就可以有大概结果，那就是**全局变量在当前进程和子 shell 都是有效的，在 fork&exec 子进程中是无效的**。
但是环境变量有一个特殊的地方，就是在子进程中也有效，这是怎么做到的呢？

其实并不复杂，首先子进程的 fork 和 exec 肯定都是执行的，那么数据就一定会被清掉，环境变量能够在子进程中有效，肯定是父进程传参给子进程的，子进程在 exec 之后还原了入参，仅此而已。
如果一个外部命令使用 C 语言写的，那么 main 函数会是这样：

```
int main(int argc, char *argv[]) {
  // 入参就是 argv
  return 0;
}
```

执行到 main 的时候，外部命令创建的进程的 exec 都已经被执行完毕了，但是在外部命令内部还是可以读到入参的。
像 *bash x.sh* 这种形式，默认帮我们做了 fork 和 exec，那么入参的读取也默认帮我们做了，所以我们能够在子进程中读到环境变量。

但是**子进程的环境变量有个局限性，就是只能在子进程内部使用，在子进程内部修改或者新增的环境变量都不会影响到父进程**。这也完全合理，可以从前面说到的进程创建来分析，他们就完全是两个进程了，当然不会互相影响。如果要影响，只能走进程间通信了。

这里还有一个点要说明一下，就是为什么很多配置脚本修改完成后，要 source 一下。比如在 bashrc 里面增加了一个 expore 环境变量后，我们会 *source ～/.bashrc*。
**source 命令就是把文本内容逐行取出来，在当前进程解释执行一遍（前面说到的 AST 深度遍历一下）**。这样新增的 expore 环境变量，就会在当前进程生效了。
这样就避免了重启终端刷新 bashrc 配置文件才能生效的耗时了。

整体下来，Shell 的运行原理没有高级语言那么多深奥的东西。核心的基础知识：**解释型语言如何解释执行**以及 **fork & exec 进程创建**。

---
