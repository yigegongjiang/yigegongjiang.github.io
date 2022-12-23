---
title: iOS log 日志系统
date: 2022-12-05 04:34:25 08:00
categories:
- 技术
tags:
- C
- iOS
- Shell
keywords:
---

printf - https://www.jianshu.com/p/7f3ae4ad439c
nslog
nslogv
syslog - https://blog.csdn.net/qq_23274715/article/details/106138885
os_log

write
calloc/alloca - 堆空间/栈空间

syslog 是一个日志系统，会将信息保存到文件中。syslog 还有后台系统，可以将日志文件进行压缩等，是一整套服务工具。printf 是调试，不会保存。
nslog/nslogv 实现了调试和保存两个功能。


CocoaLumberjack
http://cooolin.com/others/2021/01/04/ios-logging-for-console-app.html
http://silentcat.top/2017/09/06/Apple%E7%B3%BB%E7%BB%9F%E6%97%A5%E5%BF%97%E6%8D%95%E8%8E%B7%E6%96%B9%E6%A1%88/
https://juejin.cn/post/6844904151881646093#heading-7

