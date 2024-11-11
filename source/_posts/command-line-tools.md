---
title: Mac - Command Line Tools
date: 2024-11-11 11:38:32
categories:
- 技术
tags:
- iOS
- Swift
---

Mac 系统上，默认没有提供 git、xcodebuild 这些开发者命令。所以当一些终端命令(brew 等)被触发后，macos 系统会弹窗提醒用户进行【开发者工具】的安装，即【Command Lines Tools】(下面简称 tools)。
如果安装了不同版本的 xcode，则每个版本都会携带各自 xcode 版本的【tools】。
上面提到的系统弹窗，是不需要安装 xcode 也可以安装【tools】。但这个 tools 版本会比较低，内部的命令数量也会少于 xcode 携带的。即：最新版本的 xcode，携带的 tools 也是最新的。
这里就可以发现有多个 tools 目录了，如下：
1. 非 xcode 携带：`/Library/Developer/CommandLineTools`
2. xcode:`/Applications/Xcode.app/Contents/Developer`
3. xcode beta: `/Applications/Xcode-16.2.0-Beta.2.app/Contents/Developer`

在终端执行 `git` 、`xcodebuild` 的时候，一定会使用某一个 tools 中的命令，可具体使用哪一个呢？

<!-- more -->

# xcode-select

通过 `xcode-select` 可以方便的切换 main tools，默认也只有 main tools 会生效。可以通过 `man xcode-select` 查看简介。

```
xcode-select --install // 安装非 xcode 版本的 tools
xcode-select --switch <path> // 切换不同的 tools 作为 main tools
xcode-select -p // 打印当前 main tools path
```

当 tools 被指定为 `xcode-xx-beta.app/xx/Develop` 目录的时候：
在终端中执行 git 命令，最终就会使用：`/Applications/Xcode-16.2.0-Beta.2.app/Contents/Developer/usr/bin/git`
xcode-select 默认只有 1 个 tools 被激活，是操作系统级别的全局有效。
如果在 a tools 激活的同时，希望使用 b tools 该怎么办呢？在多版本 xcode 共存的时候，可能需要使用不同的 tools 命令进行开发。
这个时候在当前的命令行环境中设置`DEVELOPER_DIR` 就可以临时切换 tools 目录：

```
// 1
> xcrun --find git
/Applications/Xcode-16.2.0-Beta.2.app/Contents/Developer/usr/bin/git
// 2
export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"
// 3
> xcrun --find git
/Applications/Xcode.app/Contents/Developer/usr/bin/git
```

# xcrun

当在终端中执行 `git` 命令的时候，使用的是 tools 中的 git，这里需要探究一下。

```
// 1
> where git
/usr/bin/git
// 2
> xcrun --find git
/Applications/Xcode.app/Contents/Developer/usr/bin/git
```

通过 where 发现，`/usr/bin` 目录下是存在 git 二进制可执行文件的。为什么说执行的是 tools 中的 git 可执行文件呢？
这就要说到 xcode-select 维护多个 tools 的意义了。
`/usr/bin` 是 path 路径，可以直接找到命令。但是像 git 这些命令依托不同的 tools 环境提供，总不能在切换 main tools 的时候，把这些命令 copy 到 `/usr/bin` 中。所以对于 `/usr/bin` 中的 git、xcodebuild 等命令，在具体执行的时候都执行的是 main tools 中的可执行文件。
实现这个能力的，就是 【xcrun】。

xcrun 的目标只有一个，就是根据 xcode-select 设定的 mail tools 目录，在其下面寻找命令。可以通过 `man xcrun` 查看详细：

```
Options:
  -h, --help                  show this help message and exit
  --version                   show the xcrun version
  -v, --verbose               show verbose logging output
  --sdk <sdk name>            find the tool for the given SDK name
  --toolchain <name>          find the tool for the given toolchain
  -l, --log                   show commands to be executed (with --run)
  -f, --find                  only find and print the tool path
  -r, --run                   find and execute the tool (the default behavior)
  -n, --no-cache              do not use the lookup cache
  -k, --kill-cache            invalidate all existing cache entries
  --show-sdk-path             show selected SDK install path
  --show-sdk-version          show selected SDK version
  --show-sdk-build-version    show selected SDK build version
  --show-sdk-platform-path    show selected SDK platform path
  --show-sdk-platform-version show selected SDK platform version
```

上面示例中的 `xcrun --find git` 就是用来找 git 二进制可执行文件路径的。
当 git 在 tools 中的路径找到后，原先执行的 `/usr/bin/git` 命令，也就通过这个新的执行文件来实现了。

___


