---
title: Vim 技巧
date: 2023-01-26 23:55:42
categories:
- 工具
tags:
- Work
---

一年前写了一篇[提高效率的手艺](https://www.yigegongjiang.com/2022/workefficiency/)，提到了 Vim 作为划时代的文本编辑工具，可以有效的提高文本编辑的效率，对于写作或者 coding 都非常有用。
今天整理一个稍微进阶的教程。vimer 十几年的 95% 的操作使用，应该都在这里。

> Vim 作者，Bram 于 2023.08.05 日去世了，享年 62 岁。像我这种只要碰键盘就离不开 vim 的人，对 vim 的感激，是十分强烈的。悼念 Bram。
> 计算机的基石和发展，也就这几十年。那一批有卓越贡献的人，后面二十年会相继离去。C 语言之父丹尼斯十年前走了，70 岁。后面会越来越多。

<!--more -->
# 按键

## 单个行为操作

- 移动：hjkl / w / e / b / $ / 0(^)
- 删除：dd / x
- 行合并：J
- 换行：o / O
- 撤回 / 返回：u / C+r
- 插入：i / （I-A-a）
- 查找：f(-;-,) / t / * / #
- 替换：r
- 括号匹配：%
- 黏贴复制：yy-dd/p
- 拖屏：HLM / zz(zt-zb) / c+e / c+y / c+f(d-b-u) / :n / G / gg
- 位置标记：ma-'a(mx - moving - 'x)
- 保存退出：:wq（[https://stackoverflow.com/questions/11828270/how-do-i-exit-vim](https://stackoverflow.com/questions/11828270/how-do-i-exit-vim)）

## 4 种模式

esc：普通模式
i/a：插入模式
**shift+:：**命令模式
ctrl+v：区块模式

## 多个行为组合

动词：d / c / y / v / ...
介词：i / a / f / t / ...
名词：w / e / p / { / } / ( / )  / [ / ] / " / ' / 字符 / ...

- 动词 - (数字) - (介词) - 名词
    - dw / dt? / ci) / ca} / d2w / y$ / y3w / v2i)
- 数字 - 动词 - (名词)
    - 6ixy / 4p / 2dw / 0y$

## 寄存器

- 录制：q[a-z] - anyaction - q
- 使用录制：3@[a-z]

## 寄存器使用

- 多行文本的行首/行尾增加同样的内容
    - 使用区块：c+v -> j/h -> $/0/^ -> I/A -> input -> esc
    - 使用寄存器：qa -> $/0/^ -> I/A -> input -> esc -> j -> q | 3@a
- 多行文本，增加序号
    - :let i=0（定义变量）
    - qa（开始录制）
    - 0 -> i -> input -> esc -> 0（到行首输入需要的分割等内容，然后回到普通模式，再次回到行首）
    - :let @n=i（变量放入寄存器）
    - :let i=i+1（变量+1，供下次使用）
    - "nP（将寄存器的值复制到行首之前）
    - j（进入下一行）
    - q（结束录制）
    - 3@a（回放 3 次）

# 配置

～/.vimrc，示例如下：

[vimrc/basic.vim at master · amix/vimrc](https://github.com/amix/vimrc/blob/master/vimrcs/basic.vim)

## 插件管理器

### Vundle

[https://github.com/VundleVim/Vundle.vim](https://github.com/VundleVim/Vundle.vim)

```
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from <http://vim-scripts.org/vim/scripts.html>
" Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
" Plugin 'ascenator/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line

```

### Dein

[https://github.com/Shougo/dein.vim](https://github.com/Shougo/dein.vim)

```
sh -c "$(wget -O- <https://raw.githubusercontent.com/Shougo/dein-installer.vim/master/installer.sh>)"

```

## 插件

示例：[https://github.com/preservim/nerdtree](https://github.com/preservim/nerdtree)

```
// Vundle
call vundle#begin()
  Plugin 'preservim/nerdtree'
call vundle#end()

---
// Dein
call dein#begin()
  call dein#add('preservim/nerdtree')
call dein#end()

```

插件就 Google 尽情的搜索吧，选择自己喜欢和必须的就好。Vim 插件装多了会卡，NeoVim 不会。
插件都是程序员非商用的产出，都是没有什么美感的。有些配置还比较繁琐，使用问题不大。
就这样。

# NeoVim

和 vim 有配置上的差异，使用上一样。**推荐使用 NeoVim**。

上面的配置、插件管理器等都是 Vim 的，NeoVim 会有一些不一样，但大体也都是一致的。插件也都是能共用的，很多插件也为 NeoVim 做了专门适配。

# IDE

[https://spacevim.org/](https://spacevim.org/)

![](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202301270125293.png)

1. 多种开发语言的配置箱
2. 缓存/窗口管理方便
3. 撤销树
4. 提供的快捷帮助命令使用很方便
5. 简化配置
6. 命令终端使用非常方便

# 工作使用

## Vim 终端

- 单文件查看/编辑
- 系统文件配置
- coding

## 集成IDE

- vscode(安装 vim 插件)：全局搜索 / 代码阅读 / json格式化
- xcode(打开 vim 模式)：函数调用链 / code

# 离开鼠标

## 网页 vim 插件（chrome/safari）

![](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202301270126742.png)

![](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202301270126310.png)

## 文件浏览器 Ranger

![](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202301270126737.png)

## 全键盘 homerow

[https://www.homerow.app/](https://www.homerow.app/)

![](https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202301270126377.png)

# 写作推荐

问题：

- 频繁修改前面写过的内容，使用 Vim 可以更专注于内容。避免手的移动干扰思路。
- 中文场景下 vim 体验不好（普通模式下的键盘操作需要英文）

推荐：
**Xcode**

- xcode 官方自定义了一套 vim，适配中文场景非常棒。写作过程中基本不用再切换输入法。
- 如果需要搭建复杂文档，有多个文件和文件夹，可以通过 git 、workspace、playground 自行配置。
    - git：云存储
    - workspace：多个文件系统做隔离
    - playground：markdown 和 预览
