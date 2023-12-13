---
title: 【Swift 三方源码1】SwiftShell 高效的命令行工具
date: 2023-12-13 13:56
categories:
- 技术
tags:
- Swift
- Swift三方源码系列
keywords:
---

不推荐使用 Swift 写脚本，和 Python 比起来，该生态链想对匮乏，开发耗时会增加很多。
但对于偏 Swift 的同学来说，这也的确是更可控和方便维护的方式之一。尤其对于公司内部工具，有问题可以更快的找到原因并处理，不至于手忙脚乱。
通过 Swift 进行脚本开发的环境和入门，可参考：[Swift 脚本开发环境搭建](https://www.abc.com)

在使用脚本的时候，经常会使用到在终端环境中使用的命令、管道、文件读取等。这些能力，[SwiftShell](https://github.com/kareman/SwiftShell) 做了完备的封装，很方便使用。
这里对该库进行一些解读。

# 前置

在进行源码分析之前，需要先对 Swift 中的 文件读取 和 进程 进行该要描述，这是 SwiftShell 重点依靠的系统能力。熟悉它们之后，可以更熟悉 SwiftShell 的 Api 到底接管了哪些能力。

## 0x01 FileManager


## 0x02 FileHandle

// 标准输入输出错误
// seek
// readToEnd
// avaliabledata
// offset

## 0x03 Process

// path - bash / usr/bin/env / ...

// 之前写的 shell 环境的文章

# 源码分析

## 整体设计

## Stream 

## Context

## Command
// 四种方式

<!-- more -->



___


