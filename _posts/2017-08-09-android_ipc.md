---
layout: post
title: Android的进程是怎么相互通信的
categories: [Android]
keywords: IPC, Android
---

为什么需要解决进程间通信的问题？因为操作系统为了安全考虑把进程的内存空间相互隔离，一个进程无法读写另外一个进程的内存空间，这样的话，一个进程需要和另外一个进程通信时，就必须借助其他设施，这样就有了很多为解决进程间通信的程序出现，Binder就是其中之一。由于应用场景和侧重点不一样，这些程序的实现和使用方式也有很大的区别。

再来看看进程间通信面临什么样的困难？由于进程的内存空间是相互隔离的，所以进程A要给进程B发送消息时，必须想办法突破这种隔离，但又不能违背内存空间隔离的安全原则。如果进程A想和多个进程同时通信，怎么管理这种同时通信而不导致混乱。通信过程怎样进行才有效率。通信时其中一个进程突然终止运行需要怎么处理。在实现中会面临什么具体困难呢？进程间通信程序应该运行在内核空间中，用户空间的进程怎么访问内核空间的程序呢，当然这是内核需要考虑的问题。应该提供什么样的使用方式才会让进程感觉很自然呢。

在Linux操作系统中，要让两个内存空间相互隔离的进程相互通信，必须将目光转移到这个两个进程都能访问的公共区域，只有这样的地方才让这个两个进程交互信息成为可能，这样的公共区域有内核内存空间、磁盘文件等，从信息交互的速度来讲，由于内存的读写速度最快，所以用内核的内存空间作为两个进程彼此交互信息的场所是最好的。因此在Linux系统中，所有进程间通信程序本质上都是通过公共内存区域实现的。 

   