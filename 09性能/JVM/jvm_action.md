---
title: JVM 实战
tags: 
  - JVM
categories: 学习笔记
top: 0
copyright: ture
date: 2019-06-30 03:31:35
---

# 常用的Jvm启动参数
 - -Xmx512m：设置JVM最大可用内存为512M。
 - -Xms512m：设置JVM初始内存为512m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。<!--more-->
 - -Xmn200m：设置年轻代大小为200M。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8（young占30%左右）

 - -verbose:gc：开启gc日志
 - -Xloggc:gcc.log：将日志输出到文件xx(默认位置为桌面)
 - -XX:+PrintGCDetails：打印GC详情
 - -XX:+PrintGCDateStamps：打印GC时间戳 

参考：
[GC性能优化](https://blog.csdn.net/renfufei/column/info/14851)
[美团 - 从实际案例聊聊Java应用的GC优化](https://tech.meituan.com/2017/12/29/jvm-optimize.html)

# FGC实战
以JxlTest为例，通过gc日志，fix FGC问题。参考[阿飞的博客-FGC实战](https://mp.weixin.qq.com/s/0VeZYuoMG8PimeHk4CbjTQ)

阅读一下GC日志

