---
title: 开源清单
date: 2024-01-15 03:32:11
---

# 小工具

* [JapanMemory - 日本语速记](https://github.com/yigegongjiang/JapanMemory)： **Mac app**
> 刚开始学习日语，五十音图总是记不住。于是想到了把记不住的音定时循环随机展示，以加强记忆，效果良好。
  * Next...
    * 通过配置文件，可外部动态配置待记忆内容
    * 增加单词记忆

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202401160655916.jpg" width="30%">
<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202401160655917.jpg" width="30%">

* [HLVPaste - 轻量级文本输入框](https://github.com/yigegongjiang/HLVPaste)： **Mac app**
> 通过 LaunchBar 等启动工具，可快速打开 app HLVPaste 并获得焦点（如已经打开，上一次的内容会自动清空）。
> HLVPaste 有非常小巧的界面，输入文本后即可移动到其他窗口。文本会自动 copy 到 Mac 粘贴版。
> 在 iPhone 设备上可 Paste，获取到 Mac 的粘贴版内容。

<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202401150818590.png" width="30%">

* **中文分词**： **Mac app & CLI**
> 一直有一个写字痛点，就是错别字。尤其在发帖子和写文章这样的正式场合，错别字会引起很大的误解，而每次检查都会很吃力。
> 试用了人气较高的 “写作猫” 和 “火龙果” 两个纠错平台，都有很大的局限性，并不适合我这样的人使用。
  * [中文词语纠错说明](https://www.yigegongjiang.com/2024/SwiftZhTextCorrect/)
  * [HLVSentence CLI](https://github.com/yigegongjiang/HLVSentence)
  * [HLVZhCorrect Mac app](https://github.com/yigegongjiang/HLVZhCorrect)
  
<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202401061607920.png" width="30%"/>
<img src="https://cdn.jsdelivr.net/gh/yigegongjiang/image_space@main/blog_img/202401061607769.png" width="30%"/>

* [HLVFileDump - 文件类型识别](https://github.com/yigegongjiang/HLVFileDump)： **CLI**
> 对于一个文件是否是文本文件，并没有完全行之有效的识别方案。只能通过尝试去理解内容，这是有一定误差的。
> 可行的方案有：文件名后缀匹配、magic number 过滤等，虽然会有一定误差，但在比较稳定的环境下，这也是有效的。
> 对文本内容全量解码，这在文本较小的时候非常行之有效。若环境中出现图片、压缩文件等，这会有极大的性能损耗。
> 这里提供一种思路，即对文本内容主动进行多个位置的截取解码，以较小的性能开销来对文本文件进行识别。
```
hlvdump istext xxx.md

> YES, This Is Text File.(or "NO, This Not Text File.")

hlvdump MyI.zip
> type: zip, alias: jar zipx

hlvdump test.log
> type: text
```

# 博客 Local

当前 [博客](https://github.com/yigegongjiang/yigegongjiang.github.io) 可在 `Github - master` 中下载，可在本地进行 html 查阅。



___

枯萎的野草 怎配得上栀子花
在冬夜的我 留不住你的初夏
祝你以后过得比我好啊

\- from《野草与栀子花》
