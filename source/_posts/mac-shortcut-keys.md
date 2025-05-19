---
title: Mac 快速修改系统快捷键
date: 2025-05-19 14:14:27
categories:
  - 工具
tags:
  - Work
---

Macos 系统快速调整快捷键方案：

1. 在 `system settings` 中清理不需要的快捷键，把核心快捷键用来自定义
2. `Raycast` - 最方便的快速创建自定义快捷键解决方案
3. `Hammerspoon` - 复杂疑难病症的解决方案

操作说明和快捷键建议指南如下。

<!-- more -->

## 系统快捷键设置

入口：`system settings` - `keyboard`

- `Press 地球 key to` 调整为 nothing
  - 候选项有`Change Input Source`、`Show Emoji & Symbols`、`Start Dictation`
  - Input Source 通过 `Input Source Pro` app 可以快速设置（支持快捷键）
  - Emoji & Symbols 通过系统全局 `Control + CMD + Space` 呼出，不需要快捷键
  - Start Dictation 如果不需要这个功能，不用开启
- `Dictation` - `Shortcut`: 改成不需要常用快捷键的方式
- `Keyboard Shortcuts` - 这里面是系统提供的很多没必要的快捷键，不需要的都可以取消掉

上面三个步骤设置完成后，核心的快捷键都可以留给我们自己来分配了。

## 通过 Raycast 修改

Raycast 非常好用，通过 `Hotkey` 可以快速设置系统级快捷键，非常方便。

1. 打开 app：一般不需要，通过 Raycast 启动入口打开 app 已经很快了
2. Script Command：这个非常好用，很多系统级别的能力，可以通过 sh/applescript 的形式执行，分配一个快捷键后非常方便。
   - e.g. 写一个【当前最前置 window】移动到下一个屏幕上（多屏幕场景），增加 [Double Click CMD] 快捷键。

## 通过 Hammerspoon 修改

Hammerspoon 是非常强大的快捷键执行器,解决 any 疑难杂症。

在多屏幕场景下，很难实现【移动鼠标到指定屏幕】，Hammerspoon 可以实现。

## 快捷键建议

- 1 只手能操作的快捷键，绝不留给 2 只手
- `Ctrl`、`Option`、`Cmd` 和 `q/w/e/r/a/s/d/f/z/x/c/v` 组合
  - 尽量少用 `Shift`
- `Double Click Ctrl`、`Double Click Option`、`Double Click Cmd` 这三个最方便，要留给最常用的操作
- 很多时候快捷键操作的都是【文件/文件夹】，Mac Finder 支持太弱了，上 `QSpace Pro` 就非常方便（使用 Raycast Script Cammand 也很方便）。
- 快捷键的脚本怎么写，别自己写，让 AI 写。

---
