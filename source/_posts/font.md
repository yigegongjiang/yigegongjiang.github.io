---
title: Mac 字体
date: 2024-10-22 10:51:53
categories:
- 技术
tags:
- 计算机基础
---

> 很久之前，写过一篇关于 【[**计算机字符编码与内存编码 - Unicode**](https://www.yigegongjiang.com/2023/unicode/)】 的快照，根据 码位、码表 对字符进行了介绍。这里特别说明一下 Mac 系统上的字体库。

系统自带软件：`Font book` 
字体文件夹： `/System/Library/Fonts` 、`/Library/Fonts`、`~/Library/Fonts`
苹果提供的字体：`Applexxx` 、`Apple xxx` 、`SFxxx` 、`PingFangxxx` 

# 如何使用字体

1. 操作系统根据语言的不同，会使用默认字体。比如中文系统，会使用 `PingFang` 字体。英文系统，会使用 `SF` 字体。
2. app、dmg 等软件，可以直接使用 `defaultfont:xxx` 的形式直接使用系统字体，或者使用 `fontname:xxx` 的形式自行选择字体。自行选择的时候，可以使用系统提供的字体，也可以将 xx 字体打入 app 中来使用。
3. app 提供修改字体的功能，用户可以自行选择需要的字体。

<!-- more -->

# 图标

Apple 平台提供了两个图标字体，分别是：**`Apple Color Emoji` 和`Apple Symbols` 。**

字体库能够包含图标、颜色，在之前阐述 Unicode 的时候已经说明过，因为它们都依靠`码位` 进行检索。所以在不同的平台上，都会有自己的图标库的系统级别实现，显示效果会不一样。

# Nerd Font

nerd font 是一个项目集合，它将非常多的字体进行扩展，增加了许多 icon，从而让已经非常优秀的字体扩展了更多的功能。

可以通过 brew 安装 nerd 字体。

```jsx
> brew search nerd-font

font-0xproto-nerd-font                                font-iosevka-nerd-font
font-3270-nerd-font                                   font-iosevka-term-nerd-font
font-agave-nerd-font                                  font-iosevka-term-slab-nerd-font
font-anonymice-nerd-font                              font-jetbrains-mono-nerd-font ✔
font-arimo-nerd-font                                  font-lekton-nerd-font
font-aurulent-sans-mono-nerd-font                     font-liberation-nerd-font
font-bigblue-terminal-nerd-font                       font-lilex-nerd-font
font-bitstream-vera-sans-mono-nerd-font               font-m+-nerd-font
font-blex-mono-nerd-font                              font-martian-mono-nerd-font
font-caskaydia-cove-nerd-font                         font-meslo-lg-nerd-font
font-caskaydia-mono-nerd-font                         font-monaspace-nerd-font
font-code-new-roman-nerd-font                         font-monocraft-nerd-font
font-comic-shanns-mono-nerd-font                      font-monofur-nerd-font
font-commit-mono-nerd-font                            font-monoid-nerd-font
font-cousine-nerd-font                                font-mononoki-nerd-font
font-d2coding-nerd-font                               font-noto-nerd-font
font-daddy-time-mono-nerd-font                        font-open-dyslexic-nerd-font
font-dejavu-sans-mono-nerd-font                       font-overpass-nerd-font
font-droid-sans-mono-nerd-font ✔                      font-profont-nerd-font
font-envy-code-r-nerd-font                            font-proggy-clean-tt-nerd-font
font-fantasque-sans-mono-nerd-font                    font-recursive-mono-nerd-font
font-fira-code-nerd-font ✔                            font-roboto-mono-nerd-font
font-fira-mono-nerd-font                              font-sauce-code-pro-nerd-font
font-geist-mono-nerd-font                             font-shure-tech-mono-nerd-font
font-go-mono-nerd-font                                font-space-mono-nerd-font
font-gohufont-nerd-font                               font-symbols-only-nerd-font
font-hack-nerd-font ✔                                 font-terminess-ttf-nerd-font
font-hasklug-nerd-font                                font-tinos-nerd-font
font-heavy-data-nerd-font                             font-ubuntu-mono-nerd-font
font-hurmit-nerd-font                                 font-ubuntu-nerd-font
font-im-writing-nerd-font                             font-ubuntu-sans-nerd-font
font-inconsolata-go-nerd-font                         font-victor-mono-nerd-font
font-inconsolata-lgc-nerd-font                        font-zed-mono-nerd-font
font-inconsolata-nerd-font                            netron ✔
font-intone-mono-nerd-font
```

通过 `brew install --cask font-hack-nerd-font` 安装的字体，会被安装到 `~/Library/Fonts` 文件夹中。

终端中建议使用 `hack` 字体。

# zsh

zsh 有 `powerlevel10k` 主题。该主题在字体方面比较丰富，主要是采用了很多字体 icon。有些字体如果提供不了 所需 的 icon，显示就会异常。

但是 `powerlevel10k` 本身仅仅是配置文件，它不提供字体的安装。所以用户需要自行安装所需要的字体。它和 nerd font 配合比较友好，只要是 nerd font 项目中的字体，都可以被 `powerlevel10k` 很好的使用。

操作流程：

1. 通过 brew 安装 nerd 项目中的字体，如 hack。
2. ITerm、Warp 中选择 nerd 字体。

# 其他

安装 hack 等字体后，很多天天见面的 IDE 或者 app，就可以切换喜欢的字体了。

___


