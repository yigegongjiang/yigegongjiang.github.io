---
title: Block闭包简查记录
date: 2018-11-24 17:00:08
categories:
- 技术
tags:
- iOS
keywords: Block,闭包
---

像Block这种闭包，就是怎么用怎么爽，怎么用怎么喜欢的编码方式了。

闭包很伟大，也是各个语言都实现了的基础语法。我这边对闭包的理解，就是：**内部函数持有外部变量**。

但是各个语言都有需要注意的点，如iOS里面的循环引用，Swift里面的逃逸闭包，Python里面的闭包变量延迟定义等。

<!-- more -->

```
As a local variable:
	returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};

As a property:
	@property (nonatomic, copy, nullability) returnType (^blockName)(parameterTypes);

As a method parameter:
	- (void)someMethodThatTakesABlock:(returnType (^nullability)(parameterTypes))blockName;

As an argument to a method call:
	[someObject someMethodThatTakesABlock:^returnType (parameters) {...}];

As a typedef:
	typedef returnType (^TypeName)(parameterTypes);
	TypeName blockName = ^returnType(parameters) {...};

```

