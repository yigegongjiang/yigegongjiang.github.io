---
title: Swift struct(值类型)与class(引用类型)区别及三种传参机制深刻关系和使用
date: 2019-12-20T05:13:18.955Z
categories:
- 技术
tags:
- Swift
- iOS
keywords: Swift、sturct、class、值类型、引用类型、值传参、地址传参、引用传参、抽象
---

在Swift中，struct（值类型）和class（引用类型）的区别，不仅仅在于对象复制时候表现出来的差异，也不仅仅是构造器和继承的异同，本质上却是数据抽象层级的高低。
如果不能把值传参、地址传参和引用传参与类对象联系起来，也无法理解不同传参下对象的使用和struct、class的应用场景。

因为struct和class表现出来的是语法层面的差异，而项目使用中体现的是语义层级的差异。比如，Objective-C里面的NSString，它是引用类型，但是我们却在使用它的值语义。
```
    NSMutableString *oldname = @"hello".mutableCopy;
    NSMutableString *newname = oldname;
    newname = @"word".mutableCopy;
    NSLog(@"\n oldName:%@ \n newName:%@", oldname, newname);
 
    > oldName:hello 
    > newName:word
```
我们使用了引用类型的NSMutableString，但是newname并没有引用复制oldname，仅仅是指针地址复制了oldname。这样导致了newname并不是oldname的别名(alias)。
如果newname是oldname的别名，那么对newname的所有all操作，都会同步到oldname。
这里，newname如果改变了对象数据是可以同步到oldname，但是却不能改变oldname变量的值（oldname的存储值，即“hello”的指针地址）。
所以，这里的NSMutableString虽然是引用类型，却具有值语义。

因为编程语言概念上的模糊，下面首先介绍struct和值类型的关系。
然后重点说明值类型和引用类型的区别，这是重点，直接解释了struct和class的根本区别。
最后加一点小彩蛋，介绍Swift里面struct特性。

<!-- more -->

### 0X00 struct是什么？好像在哪里见过！struct和值类型是什么关系？
好久好久之前，是没有class什么事的。那时候C语言活跃于各种应用场景，在上层抽象了汇编的实现，C语言可以用于嵌入式、服务器、驱动等。为什么没有class什么事情？class是类，是OOP（面向对象编程/对象导向编程）的专属。而C语言是过程式语言，没有对象概念，也就没有class什么事情。
那时候，都用结构体，也就是struct。如下：
```
// C语言版本
// 人员结构体，具有姓名和年龄
struct Person {
    char *name;
    int age;
};
```
那时候，这些数据都叫做**值类型**，而struct就是值类型的。值类型是**变量直接包含值**统称，其他的还有基础数据类型，如Int，long，char，bool，以及自定义struct（如上面的Person）。
后面大家就知道了，出现了C++/Java/C#等OOP语言，class开始活跃于千万家。更甚之，在某些语言上，值类型已经被淡化了。
```
// Objective-C版本 .h
@interface Person : NSObject
@property(nonatomic,copy) NSString *name;
@property(nonatomic,assign) NSInteger age;
@end
```
以上为OC的版本，我们会这么描述它：我们创建了一个Person类。他具有名称属性和年龄属性。
而且，我们也确定，如果我们新建一个Person对象,如`Person *p = Person.new;`，那么这个对象一定存放在堆内存。那我们为什么不能把对象创建在栈内存呢？
我们可以知道，上面OC创建的是Class，对象存在于堆中。那我们可不可以把对象放在栈上？至少在Objective-C开发过程中，肯定不会想到这个，因为struct/结构体/值类型被淡化了。（Java也是一样，号称纯OOP语言）
其实我们可以创建栈上的数据的，如局部变量中的Int、long等，都是在栈上的。说一个少见的，CGRect、CGSize也是在栈上的。源码实现如下：
```
/* Points. */
struct
CGPoint {
    CGFloat x;
    CGFloat y;
};
typedef struct CG_BOXABLE CGPoint CGPoint;

/* Sizes. */

struct CGSize {
    CGFloat width;
    CGFloat height;
};
typedef struct CG_BOXABLE CGSize CGSize;

/* Rectangles. */

struct CGRect {
    CGPoint origin;
    CGSize size;
};
typedef struct CG_BOXABLE CGRect CGRect;
```
其实OC中使用的CGRect等，也就是struct值类型数据。只是我们很少顾及值类型，虽然一直用，但是感知不到他们的存在。
直到有一天，Swift也成了苹果开发的官方语言，引入了struct，一大堆开发人员开始迷惑，发生了什么事？我该怎么办？
```
// Swift struct版本
struct Person {
	var name: String
	var age: UInt8
}

// Swift class版本
class Person {
	var name: String
	var age: UInt8
}
```
同样的Person，一个是struct，一个是class。一个复制的时候是全部数据复制，一个复制的时候是指针复制。一个有可能存放在栈上，一个只能存放在堆上。
放心了很多，原来struct和Int、BOOL等是一样的。他们本身不可以改变，只能被复制。struct拥有和Int一样的外观。
等等，Swift的struct可以引入方法，可以实现构造器，甚至可以通过mutating来改变自己。俨然已经上升可以和class平起平坐了（其实没有高低之分）。这怎么和Int又不太一样了？struct到底是啥？

其实struct一直都没有消失。它一直在我们周边。仅仅是因为我们在个别面向对象语言的冲击下，淡化了值类型。
而现在，在Swift中，我们必须捡起来，因为值类型在Swift中不可或缺了。甚至Swift里面的Int,String等，都是通过struct直接实现的。
```
/// A signed integer value type.
///
/// On 32-bit platforms, `Int` is the same size as `Int32`, and
/// on 64-bit platforms, `Int` is the same size as `Int64`.
public struct Int : FixedWidthInteger, SignedInteger {

    /// A type that represents an integer literal.
    public typealias IntegerLiteralType = Int
	...
	...
}
```
相比之下，C++一直存在值类型，而且C++里面的自定义的struct还可以继承和多态，完全面向对象化了。

### 0X0 值类型和引用类型的内存差异
值类型和引用类型，从语法上还是毕竟容易理解的。如果牵涉到语义，就毕竟复杂，因为上面说到的，引用类型就牵涉到了值语义。
我们先从语法上来理解值类型和引用类型的内存差异。

##### 引用类型内存图
首先，class是引用类型，一定是存放在堆内存上的。如下：
```
// Swift class版本
class Person {
    var name: String?
    var age: UInt8?
}

var person = Person(name: "Gongjiang", age: 20)
person.name = "Hello"
 
var someOne = person
someOne.name = "Robot"
 
> person.name:Robot
> someOne.name:Robot
```
那么我们定义一个Person对象时候的内存分布如下图：
[]()
person这个引用变量，我们使用的时候，可能存储在栈中的（也可能在堆中，但不是重点）。但是其指向的对象，却一定是在堆中的。
> 这里我们有两个名词认知不要弄混了，一个是person变量，一个是person对象。变量只是符号，编译的时候存储于符号表中的一个标记，对象才是我们使用的数据实体，变量用于找到对象。someOne也是同理。下文中的变量和对象都是同理，后面不再做强调。

上面操作完成后，person和someOne的name都变成Robot了，这是合情合理的，我们都司空见惯了，不做多描述。
##### 值类型内存图
下面看看值类型，
```
// Swift struct版本
struct Person {
    var name: String
    var age: UInt8
}

var person = Person(name: "Gongjiang", age: 20)
person.name = "Hello"
 
var someOne = person
someOne.name = "Robot"

> person.name:Hello
> someOne.name:Robot
```
内存分布如下图：
[]()
我们可以发现，值类型相比引用类型，person和someOne这两个值变量，**指向**特征不那么明显了，更多的是复制。我们的someOne并没有指向person，而是把person的数据完全复制了一份成为自己的。
更重要的，person和someOne存储的，**不再是对象的指针，而是真真实实的数据了**。其实对象依旧还是对象，变量依旧是变量，如上面说的那样。但是，**值变量，直接包含数据(值类型的定义)**了，不再通过指针指向数据了。
当然，我们从图上看到的，值对象是在栈里面。当然值对象也会在堆里面，场景不一样，存储位置也会不一样。但是和引用对象不同的一点，**值对象是可以存储在栈里面的**。
> 我们经常使用的Int等，其实很多时候都是存储在栈里面的。上面Person引用类，其实也有age属性，这个age属性就是值对象存储在堆上，因为person对象是在堆上的。后面说到struct和class联动内嵌的时候，会详细说明

##### 值类型和引用类型内存比较
显而易见，内存方式的不同，带来的优缺点也是迥异的。
最直接的，栈肯定是比堆快的。下面我们默认**值对象存储在栈中，引用对象存储在堆中**进行分析。
**一来**，值类型通过值变量直接定位数据，而引用类型需要通过引用变量的指针间接获取，在寻址方面就会出现时间劣势。
**再者**，栈通过CPU提供的指令寄存器操作数据，而堆通过操作系统支持。堆空间由程序员自行控制，包括垃圾回收等，CPU不会干预过多。
**其次**，我们在栈上分配内存，是直接分配。而对于引用类型，只在栈上分配变量地址，对象需要另外分配堆内存空间。（可能会出现这种情况，需要100个对象，栈类型会在栈内存中直接分配完毕，而引用类型会在栈上一次性分配100个变量内存，然后在堆中需要进行100次对象内存分配。）
**而且**，由于堆内存是空间不连续性的（操作系统分配堆内存池供开发使用，如果一个对象销毁了，就会产生内存碎片），不连续堆空间会违背局部性原理，会增加高速缓存错失的几率（命中不了）。堆空间的高速缓存指的是二级缓存，而栈是依据“LIFO”的存储机制，不会出现内存碎片，天然增加了一级缓存的命中率。
**特别**，各种语言都会着重优化值类型，以达到更快的速度。毕竟栈处理速度的快慢，直接影响到程序的快慢。因为我们的代码运行根本依靠函数，而函数就是在栈中执行，如main函数和自定义函数。所以才会加入寄存器这样的快速单元，而堆里的数据，是可以通过异步来完成存储的。
因为值对象也是会存储在堆中，所以我们可以这样说：**值类型有可能利用栈的优势，进一步提高程序的性能**。

内存比较上，相比来说，值类型完胜引用类型。
引用类型相应的优点，
一来，发生在拷贝的过程中。值类型拷贝是完全拷贝，所有数据都会拷贝一遍，而引用类型，拷贝的仅仅是指针，从而提交效率。
二来，栈内存空间有有限的，相比堆内存空间，简直太小来，如著名的网站“Stack Overflow”的名字一样，动不动就会栈溢出。而堆内存，就是普通点的服务器，4G容量不是问题的，高级点的服务器都是几十几百G容量。
我们可以看出，栈和堆相比，强于速度，弱于空间。虽然速度完胜，但除了空间，还有其他方面，引用类型却又是完胜值类型的。从Java，Objective-C这些语言上不难看出，引用类型的确是完胜值类型的。下面我们一点点来分析他们之间的不同和场景应用。

### 0X0 值类型和引用类型的相等性差异

##### 值类型相等性比较
我们回顾一些最基础的值类型Int的相等性比较。
```
var num1 = 100
var num2 = 100
assert(num1 == num2)
var num3 = 200
assert(num1 != num3)
```
上面代码编译运行，断言是可以通过的。值类型的比较，特别简单，就是比较数据是否一样。
因为值类型，不管存储在哪里，不变的一点是，值变量直接包含值对象。我们自定义一个值类型来看一下：
```
// Swift struct版本
struct Person: Equatable {
    var name: String
    var age: UInt8
}

var person = Person(name: "Gongjiang", age: 20)
person.name = "Hello"
 
var someOne = person
someOne.name = "Hello"
// someOne.name = "Robot"
 
assert(person == someOne)
// assert(person != someOne)
```
它们的内存图如下：
[]()
当两个值对象的name都是Hello的时候，两个值对象是相等的。如果someOne的name变成Robot，两个值对象就是不想等的。（如果我们把注释的代码打开，相应注释位置上一行代码删除，会发现两个断言都是可以通过的。）
值类型的相等性比较真的非常简单，就是匹配字节码是否一致。只要值对象的所有字节码是一致的，那两个值对象就是相等的。字节码从哪里来？Hello和Robot，计算机不认识的，他们都会变成对应的码值然后转化成二进制存储内存中。所以值类型相等性判断就是查看二进制是否一样。

##### 引用类型相等性比较
因为引用变量存储的是引用对象的内存地址。同样一个对象，可能有两个引用变量存储着其地址。
所以引用类型的比较，有两个方面，一个是比较存储的内存地址是否一致，另一个是比较内存地址对应的数据是否一致。
因为字符串在Swift里面是值类型的，我们用Objective-C里面的NSString来分析。
```
    NSString *str1 = [NSString stringWithFormat:@"%@", @"Hello"];
    NSString *str2 = @"Hello";
    
    assert(str1 == str2);   // ERROR
    assert([str1 isEqualToString:str2]); // OK
```
相应的内存图如下：
[]()
因为Objective-C对于字符串的生成比较考究（Objective-C里面，字符串根据创建的形式不同和存储中英文的不同，有常量区、栈区、堆区不同表现形式），我们用上面方式建立两个不同地址的Hello字符串。其中比较相等性，一个是通过**==**，一个是通过**equal**。
引用类型的相等性比较，直接通过值类型的==比较的化，比较的是内存地址，显然str1和str2，他们的内存地址不可能一样，所以他们并不相等。
而通过equal来比较，就变成了上面的值类型的字节码比较，Hello的二进制存储都是一样的，他们就相等了。

### 0X0 值类型和引用类型在花式传参过程中的异同

### 0X0 值语义和引用语义的联动性

### 0X0 值类型和引用类型的抽象层级差异

### 0X0 为什么class可以继承和多态，而struct显得很孤单？

### 0X0 struct和class联动内嵌下的认知

### 0X0 Swift中的struct为什么很特别？struct能给我们带来哪些认知？

### OX0 Swift下String的搅局误区