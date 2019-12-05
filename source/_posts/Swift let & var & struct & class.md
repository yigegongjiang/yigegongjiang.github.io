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
对于let和var，struct和class，分成两类来说，很多人比较容易理解。let：不可变，var：可变，struct：栈空间，class：堆空间。
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

上面定义了一个struct，一个class，我们下面的代码会有什么结果？

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

这里我们定义了四个对象变量，两个let，两个var。两个let里面一个是struct对象，一个是class对象。两个var也是一样。然后struct里面有一个let和一个var属性，class也是一样。

上面的代码会报错，你知道的。但是你知道会是什么样的报错吗？

![](/images/swift_let_var_1.png)
如上图所示，
同样的let对象变量(test1/test3)，struct和class有不同的错误。
同样的struct，let和var也有不同的错误。

重点来说，
1. 如果我们用let声明了一个struct对象，那么基本可以确定内部属性一个都不能改变了（不管内部属性是用let或者var修饰）。如果我们用let声明一个class的对象，那么内部属于var的属性还是可以改变的。
2. 如果我们用var声明一个struct对象，那么表现同class对象一样。

通过上面两点，我们可以发现，struct和class，他们之间的差别，肯定不仅仅是栈和堆内存的关系，不是吗？

**实际上，struct比class有更严格的约束。而let和var的可变和不可变也有更深的原理。**

## 0x02 let和var到底是什么？

如果理解C里面的指针和const不变量，那么非常容易理解了。
我们在C里面定义下面四个变量，
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

这四个常量age的存储，如下图所示：
![](/images/swift_let_var_2.png)

对于age1，我们就不能通过**age1 = 0;**这样的语句修改值了。
对于age2，因为const标记的是值，所以***age2 = 0;**是不行的，会报错。但是我们可以通过**age2 = &other;**这样的语句修改age2对应的指针值。
对于age3，因为const标记的是指针，所以**age3 = &other;**是不行的，会报错。但是我们可以通过***age3 = 0;**这样的语句修改age3对应的数值。
对于age4，因为数值和指针都是只读的，都是常量，什么也修改不了。

我们发现，其实Swift里的let，就是C里的const。**但是在struct和class中表现是不一致的。**struct类型的let变量就是**“（4）int temp = 24;const int * const age4 = &temp;”**类型，什么都修改不了。而class类型的let变量，就是**“（3）int temp = 23;int * const age3 = &temp;”**类型，本身对象指针无法修改，但是内部属性在修饰符允许的情况下是可以修改的。

我们通过上面的报错信息验证一下刚才的观点。
在代码的第15行，**test1.name2 = "t"**报错为**Cannot assign to property: 'test1' is a 'let' constant**，代码的第16行，**test1 = Person1()**报错为**Cannot assign to value: 'test1' is a 'let' constant**。他们的报错原因完全相同，而**name2**本身是var属性。所以15行和16行的报错不是struct内部决定的，而是由**let test1**这个**let**属性决定的。
同样的，我们看到代码第25行，**test3.name2 = "t"**没有报错，和第15行出现报错对比，可以确定同样的**let**类型的对象变量，在修饰struct和class的时候，他们的行为却是不一致的。

（我相信有不少朋友，这里会饶进入一个洞里，那就是C里面的const修饰的是Int基础类型，而这里let修饰的是struct对象，struct可不是Int或者String，struct可是对象啊！他们是否有可比性？如果没有可比性，那么上面的结论是否还能成立？这里做说明：Int、Char是啥？在内存里面就是一块存储区域。struct呢？也是一块存储区域。对象是啥？是方便我们开发所汇总出来的一套编程思想，即面向对象编程。过了IDE，解析器和编译器还会管你是Int基础类型或者对象类型？不会管的。`在广东人面前，什么都是肉，即使你是福建人。`退一万步来说，Swift里面，Int、String也是通过struct来实现的啊😂）

那我们在看看**var**。test2和test4两个var变量，虽然一个是struct对象，一个是class对象，但是他们的表现却是一致的。其实var就是**非 let**修饰符。let修饰符和const类似，那么var其实就是省略了const的修饰符。我们举下面例子
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
我们还可以发现，（2）和（3）其实是等价的。而（3）仅仅是省略了**const**修饰符，所以苹果一开始发布Swift语言的时候，开始完全可以指定下面的规则：
>> Swift文档官方指南：
>> 在Swift中，let表示不可变。如果什么都不写，那么默认可变。如:
>> **const age = 10**和**age = 10**
>> 编译器会根据语意自动识别age为Int类型，所以您无需显示添加Int，如：**const age: Int = 10**及**age: Int = 10**。

苹果没有这么做，或许是var和let一起写出来的代码，一来易于识别，二来看起来又有美感。而且天下语言基础语法一大抄，JS也是同时有let和var。而Dart就是有var和const。

## 0x03 struct和class的区别到底是什么？
网上已经能够看到很多两者的区别，如栈，堆，继承等。
这里，我想从属性约束上说一点，这点网上说的不多。

对于class，我们都是非常熟悉的了，毕竟面向对象编程这么久，也无需多言。属性是var，那么就是完全可变的。如果是let，那么回到上一个论题。
而struct，就毕竟陌生了。我们看下面的代码截图：
![](/images/swift_let_var_3.png)
我们发现，在struct里面，不管属性是let还是var，方法（内部函数）都是无法直接修改该属性的。如代码行10和13行所示。
如果我们需要修改内部属性，需要在方法前面加上**mutating**关键字。这在class里面是不存在的，说明struct相比class，属性约束要强一些。
当然，我们看代码16行，对于let属性，**mutating**也无能为力，这个和class倒是一样的。毕竟let本身就是完全不可变的。

所以，我们得出一个结论，struct相比class来说，在属性约束上，是要强一些的。

## 0x04 一个蹊跷点

这里给大家看一个蹊跷的地方，如下图所示：
![](/images/swift_let_var_4.png)
上面的代码，编译和运行都不会有问题。
我们来分析一下问题点：
1. 对于Person1内部来说，var修饰的name2属性，是不能随便改变的。如果要改变，必须要通过func前面添加mutating来实现。本质上这是一种内部约束。
2. 对于var修饰的test2对象来说，可以直接通过变量修改name2属性。就是说，可以越过struct内部约束直接修改变量值。

这里的蹊跷点就在于，外部约束大于内部约束了。我感觉不可思议，但这的确是一个事实。

## 0x05 总结

1. Swift里的let，就是C里的const。**但是在struct和class中表现是不一致的。**struct类型的let变量就是**“（4）int temp = 24;const int * const age4 = &temp;”**类型，什么都修改不了。而class类型的let变量，就是**“（3）int temp = 23;int * const age3 = &temp;”**类型，本身对象指针无法修改，但是内部属性在修饰符允许的情况下是可以修改的。
2. struct相比class，内部属性约束会更强一些。

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