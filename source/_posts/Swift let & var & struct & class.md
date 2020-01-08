---
title: Swift let &amp; var &amp; struct &amp; class
date: 2019-12-05T10:54:13.648Z
categories:
- 技术
tags:
- Swift
keywords:
---

## 0x01 问题描述和表现
对于 let 和 var，struct 和 class，分成两类来说，很多人比较容易理解。let：不可变，var：可变，struct：栈空间，class：堆空间。
当我想要确认他们的原理的时候，截止发文为止，我没有在中文互联网上搜索到相关信息。所以我把他们的原理写在下面。

下面的示例，你能够理清楚多少？
```
struct Person1 {
    let name1: String = "name1"
    var name2: String = "name2"
}

class Person2 {
    let name1: String = "name1"
    var name2: String = "name2"
}
```

上面定义了一个 struct，一个 class，我们下面的代码会有什么结果？

<!-- more -->

```
let test1 = Person1()
test1.name1 = "t"
test1.name2 = "t"
test1 = Person1()

var test2 = Person1()
test2.name1 = "t"
test2.name2 = "t"
test2 = Person1()

let test3 = Person2()
test3.name1 = "t"
test3.name2 = "t"
test3 = Person2()

var test4 = Person2()
test4.name1 = "t"
test4.name2 = "t"
test4 = Person2()
```

这里我们定义了四个对象变量，两个 let，两个 var。两个 let 里面一个是 struct 对象，一个是 class 对象。两个 var 也是一样。然后 struct 里面有一个 let 和一个 var 属性，class 也是一样。

上面的代码会报错，你知道的。但是你知道会是什么样的报错吗？

![](/images/swift_let_var_1.png)
如上图所示，
同样的 let 对象变量(test1/test3)，struct 和 class 有不同的错误。
同样的 struct，let 和 var 也有不同的错误。

重点来说，
1. 如果我们用 let 声明了一个 struct 对象，那么基本可以确定内部属性一个都不能改变了（不管内部属性是用 let 或者 var 修饰）。如果我们用 let 声明一个 class 的对象，那么内部属于 var 的属性还是可以改变的。
2. 如果我们用 var 声明一个 struct 对象，那么表现同 class 对象一样。

通过上面两点，我们可以发现，struct 和 class，他们之间的差别，肯定不仅仅是栈和堆内存的关系，不是吗？

**实际上，struct 比 class 有更严格的约束。而 let 和 var 的可变和不可变也有更深的原理。**

## 0x02 let 和 var 到底是什么？

如果理解 C 里面的指针和 const 不变量，那么非常容易理解了。
我们在 C 里面定义下面四个变量，
```
（1）
const int age1 = 21;
```
```
（2）
int temp = 22;
const int *age2 = &temp;
```
```
（3）
int temp = 23;
int * const age3 = &temp;
```
```
（4）
int temp = 24;
const int * const age4 = &temp;
```

这四个常量 age 的存储，如下图所示：
![](/images/swift_let_var_2.png)

对于 age1，我们就不能通过**age1 = 0;**这样的语句修改值了。
对于 age2，因为 const标记的是值，所以***age2 = 0;**是不行的，会报错。但是我们可以通过**age2 = &other;**这样的语句修改 age2 对应的指针值。
对于 age3，因为 const标记的是指针，所以**age3 = &other;**是不行的，会报错。但是我们可以通过***age3 = 0;**这样的语句修改 age3 对应的数值。
对于 age4，因为数值和指针都是只读的，都是常量，什么也修改不了。

我们发现，其实 Swift 里的 let，就是 C 里的 const。**但是在struct和class中表现是不一致的。**struct 类型的 let 变量就是**“（4）int temp = 24;const int * const age4 = &temp;”**类型，什么都修改不了。而class类型的let变量，就是**“（3）int temp = 23;int * const age3 = &temp;”**类型，本身对象指针无法修改，但是内部属性在修饰符允许的情况下是可以修改的。

我们通过上面的报错信息验证一下刚才的观点。
在代码的第 15 行，**test1.name2 = "t"**报错为**Cannot assign to property: 'test1' is a 'let' constant**，代码的第16行，**test1 = Person1()**报错为**Cannot assign to value: 'test1' is a 'let' constant**。他们的报错原因完全相同，而**name2**本身是 var 属性。所以 15 行和 16 行的报错不是 struct 内部决定的，而是由**let test1**这个**let**属性决定的。
同样的，我们看到代码第 25 行，**test3.name2 = "t"**没有报错，和第 15 行出现报错对比，可以确定同样的**let**类型的对象变量，在修饰 struct 和 class 的时候，他们的行为却是不一致的。

（我相信有不少朋友，这里会饶进入一个洞里，那就是 C 里面的 const 修饰的是 Int 基础类型，而这里 let 修饰的是 struct 对象，struct 可不是 Int 或者 String，struct 可是对象啊！他们是否有可比性？如果没有可比性，那么上面的结论是否还能成立？这里做说明：Int、Char是啥？在内存里面就是一块存储区域。struct 呢？也是一块存储区域。对象是啥？是方便我们开发所汇总出来的一套编程思想，即面向对象编程。过了 IDE，解析器和编译器还会管你是 Int 基础类型或者对象类型？不会管的。`在广东人面前，什么都是肉，即使你是福建人。`退一万步来说，Swift 里面，Int、String 也是通过 struct 来实现的啊😂）

那我们在看看**var**。test2 和 test4 两个 var 变量，虽然一个是 struct 对象，一个是 class 对象，但是他们的表现却是一致的。其实 var 就是**非 let**修饰符。let 修饰符和 const 类似，那么 var 其实就是省略了 const 的修饰符。我们举下面例子
```
(1)
const int age = 10;
```
```
(2)
not const int age = 10;
```
```
(3)
int age = 10;
```
上面例子中，**not const**修饰符是我杜撰的。意思就是**非const**。那么我们可以发现，**not const**就是**var**。
我们还可以发现，（2）和（3）其实是等价的。而（3）仅仅是省略了**const**修饰符，所以苹果一开始发布 Swift 语言的时候，开始完全可以指定下面的规则：
>> Swift 文档官方指南：
>> 在 Swift 中，let 表示不可变。如果什么都不写，那么默认可变。如:
>> **const age = 10**和**age = 10**
>> 编译器会根据语意自动识别 age 为 Int 类型，所以您无需显示添加 Int，如：**const age: Int = 10**及**age: Int = 10**。

苹果没有这么做，或许是 var 和 let 一起写出来的代码，一来易于识别，二来看起来又有美感。而且天下语言基础语法一大抄，JS 也是同时有 let 和 var。而 Dart 就是有 var 和 const。

## 0x03 struct 和 class 的区别到底是什么？
网上已经能够看到很多两者的区别，如栈，堆，继承等。
这里，我想从属性约束上说一点，这点网上说的不多。

对于 class，我们都是非常熟悉的了，毕竟面向对象编程这么久，也无需多言。属性是 var，那么就是完全可变的。如果是 let，那么回到上一个论题。
而 struct，就毕竟陌生了。我们看下面的代码截图：
![](/images/swift_let_var_3.png)
我们发现，在 struct 里面，不管属性是 let 还是 var，方法（内部函数）都是无法直接修改该属性的。如代码行 10 和 13 行所示。
如果我们需要修改内部属性，需要在方法前面加上**mutating**关键字。这在 class 里面是不存在的，说明 struct 相比 class，属性约束要强一些。
当然，我们看代码 16 行，对于 let 属性，**mutating**也无能为力，这个和 class 倒是一样的。毕竟 let 本身就是完全不可变的。

所以，我们得出一个结论，struct 相比 class 来说，在属性约束上，是要强一些的。

## 0x04 一个蹊跷点

这里给大家看一个蹊跷的地方，如下图所示：
![](/images/swift_let_var_4.png)
上面的代码，编译和运行都不会有问题。
我们来分析一下问题点：
1. 对于 Person1 内部来说，var 修饰的 name2 属性，是不能随便改变的。如果要改变，必须要通过 func 前面添加 mutating 来实现。本质上这是一种内部约束。
2. 对于 var 修饰的 test2 对象来说，可以直接通过变量修改 name2 属性。就是说，可以越过 struct 内部约束直接修改变量值。

这里的蹊跷点就在于，外部约束大于内部约束了。我感觉不可思议，但这的确是一个事实。

## 0x05 总结

1. Swift 里的 let，就是 C 里的 const。**但是在struct和class中表现是不一致的。**struct 类型的 let 变量就是**“（4）int temp = 24;const int * const age4 = &temp;”**类型，什么都修改不了。而 class 类型的 let 变量，就是**“（3）int temp = 23;int * const age3 = &temp;”**类型，本身对象指针无法修改，但是内部属性在修饰符允许的情况下是可以修改的。
2. struct 相比 class，内部属性约束会更强一些。

下面给出我这边完整测试代码（可通过代码右上方复制按钮直接复制）：

```
import Foundation

struct Person1 {
    let name1: String = "name1"
    var name2: String = "name2"
    let arr1: [Int] = []
    var arr2: [Int] = []
    
    func change1() {
        name1 = "test"
    }
    func change2() {
        name2 = "test"
    }
    mutating func change3() {
        name1 = "test"
    }
    mutating func change4() {
        name2 = "test"
    }
}

class Person2 {
    let name1: String = "name1"
    var name2: String = "name2"
    let arr1: [Int] = []
    var arr2: [Int] = []
    
    func change1() {
        name1 = "test"
    }
    func change2() {
        name2 = "test"
    }
    mutating func change3() {
        name1 = "test"
    }
    mutating func change4() {
        name2 = "test"
    }
}

let test1 = Person1()
test1.name1 = "t"
test1.name2 = "t"
test1.arr1.append(1)
test1.arr2.append(1)
test1 = Person1()

var test2 = Person1()
test2.name1 = "t"
test2.name2 = "t"
test2.arr1.append(1)
test2.arr2.append(1)
test2 = Person1()

let test3 = Person2()
test3.name1 = "t"
test3.name2 = "t"
test3.arr1.append(1)
test3.arr2.append(1)
test3 = Person2()

var test4 = Person2()
test4.name1 = "t"
test4.name2 = "t"
test4.arr1.append(1)
test4.arr2.append(1)
test4 = Person2()

```

___

大千世界，真的无奇不有。很多自然科学无法解答的问题，都是奇闻逸事。
如果有人说自己观察到尸体动了一下，但是本该绝对不可能动。这怎么用科学来解释？说这个人撒谎？但是这是一个正直刚正的人，又绝对不可能撒谎！
我老家的老人们说自己看到了已经离开了很久的村里人在路上走路，这又该如何解释？他们可能撒谎了，也可能没有撒谎。
我们可以用压力过大、悲伤过度、思念强烈等客观原因来分析，但也没有科学依据来证实这些客观原因能够导致上面的现象。
世界本就是无奇不有，对我们有恶意的，我们不要贸然进入。对我们没有恶意的，我们也无需害怕和担忧。
海底的生物有多奇特，人类不知道。地下深处结构如何，人类也未可知。
很多人认为宇宙飞船都上天了，在这么发达的社会，无法用科学来解释的问题，都是恐怖的。
但是科学才触及到真理的哪个边界啊！或许科学连真理的边缘还没有触及到呢。
当前人类最应该害怕的，难道不应该是人类本身么？