---
title: rime - 小鹤双拼
date: 2023-09-21 18:47:49
categories:
- 工具
tags:
- Life
- Work
keywords: rime,squirrel,双拼,小鹤双拼,配制
---

> 2023.10.03 更：采用 twitter 网友的办法，我又换回系统输入法了。详见：https://x.com/hai_lv_/status/1704597086346649778?s=20

Mac 上原生的双拼输入法，有时候间歇性的卡死。在网上找了不少办法，均不能很好的解决。
我对原生输入法还是有不少感情的。自从 Mac 和 iPhone 支持小鹤双拼后，就迁移到了原生输入法，到目前也有很多年了。

这次把 Mac 侧迁移到 Rime，也尽量和原生的体验保持一致。鉴于 Rime 本身配置较为复杂，这里把自己使用的精简版做下记录。

体验说明（和 原生 的输入方式保持一致，包括中英文切换方式）：
1. 使用 CapsLock 键作为 英文 和 中文 的切换键。
2. Rime 仅仅有 **小鹤双拼** 的输入能力，其他输入法包括英文均做了阉割。
3. 各种类型的符号输入，和原生保持一致。还有 半角、全角、生僻字、Shift 快捷键 等。

基本上，就是无感切换了。

配置，都已经处理好了。真要配置这些细节，还是挺花费时间的，我前后共计花了 4-5 小时。
如果你和我的输入法使用习惯一致，建议直接拿去用，不要想着二次配置了。毕竟这只是工具。浪费时间不值得。
预计 10 分钟以内，可以完成本文配制，顺利使用 Rime 的小鹤双拼。

<!-- more -->

# 0x1

brew 安装 输入法
`brew reinstall --cask squirrel`

然后退出当前用户重新登录，或者重启电脑。

现在可以在设置里面找到 Squirrel 输入法了。

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202309220148970.png" width="30%">

完成后，就可以通过 CapsLock 键切换输入法了。这里希望你也是这样的用户习惯，不然后面的配置可能不适用。因为有些同学习惯 Shift 切换输入法。
<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202309220148972.png" width="30%">

# 0x2

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202309220148971.png" width="30%">

点击设置，会打开 输入法 的配置文件夹。这里下载配置文件，全部复制到当前配置文件夹里面。https://github.com/yigegongjiang/rime

然后点击上面截图里面，setting 上面有一个 **Deploy** 部署按扭，点击后等 3-5 秒中，配置就部署完成了。

# 0x3

目前已经可以正常是用 rime 的双拼了。enjoy yourself.

如果正常的话，应该可以出现下面两种不同模式(正常/暗黑)下的输入项，基本和 原生 的还原。
其中暗黑模式无法做到 100% 还原，因为系统会根据输入窗口的配色，对输入面版做颜色调整，但是 Rime 只能做固定配色。

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202309220148974.png" width="30%">

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202309220148973.png" width="30%">


# 进阶说明

在 rime 输入法下，按 `control+~` 键，光标出会出配制项。这里可以做进一步的配制，如启用 utf8 打开生僻字，打开 emoji，切换繁体和简体等。

如果这里满足不了诉求，可以修改下面的配制项，重新部署一下即可。这里有一份快速的配置指南。

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202309220148975.png" width="30%">

主要配制都是在 `double_pinyin_flypy.custom.yaml` 文件中，这是双拼的配制文件。

## 打开生僻字
默如和原生的逻辑一致，是关闭生僻字的。这样可以有效的减少侯选字的数量。如果需要打开生僻字，可以做如下操作：
```
    - options: ["gbk","utf8"] # 这里是屏蔽生僻字用的。不然候选里面有很多生僻字。UTF8 会打开生僻字，GBK 不会。 
      reset: 0                                          
      states:
        - GBK
        - UTF-8

```
把 reset 修改成 1

## 打开 emoji
一定要在打开 生僻字 的基础上，才能打开 emoji。
```
    - name: emoji_suggestion
      reset: 0
      states: [ "N", "Y" ] # 是否需要 Emoji。注意，只能在 选中 UTF8 的时候，才可以打开 emoji，否则有概率系统 crash(实测结果)。
```
把 reset 修改成 1

## 打开 繁体字

```
    - name: simplification
      reset: 1
      states: [ 繁, 简 ] # 繁体、间体切换。默认繁体，这里需要通过 reset 强制简体
```
把 reset 修改成 0

## 关闭 词库
```
  #載入朙月拼音擴充詞庫
  #'translator/dictionary': luna_pinyin.extended
  'translator/preedit_format': {}
```
把 `translator/dictionary` 这一行注释掉即可。下一行千万不能注释，否则双拼输入的时候，输入框会显示全拼字符。

## 彻底关闭 emoji
全局搜索一下，把 `emoji_suggestion` 相关的，都删掉即可

## 自定义表情和符号
```
  # 自定义符号上屏
  punctuator:
    import_preset: symbols
    symbols:
      "/bq": [😀,😁,😂,😃,😄,😅,😆,😉,😊,😋,😎,😍,😘,😗]
    half_shape:
      "#": "#"
      "*": "*"
      "`": "`"
      "~": "~"
      "@": "@"
      "=": "="
      '\': "、"
      "%": "%"
      "$": "¥"
      "|": "｜"
      "/": "/"
      "'": { pair: ["「", "」"] }
      "[": "【"
      "]": "】"
      "<": "《"
      ">": "》"
```
修改这里，可以自定义键盘对应的符号，也可以自定义表情。

## 自定义快捷语句

在 `custom_phrase.txt` 文件夹中，可以把需要的快捷短句录入进去。
切记，一定要用 tab 做间隔，不能直接使用空格。（文件顶部有描述）

___

GFW 不仅可以控制流量的出口，也可以反向控制流量的入口。
我们看到的外面不是真实的，外面看到的我们也不是真实的。
