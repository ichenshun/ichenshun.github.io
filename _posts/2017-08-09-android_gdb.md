---
layout: post
title: GDB调试Android程序
categories: [Android]
keywords: Android, GDB
---

从Android源码中prebuilts/misc目录下选择对应平台的gdbserver文件复制到手机上，比如arm平台就选择android-arm/gdbserver/gdbserver等，可能不能版本的android源码的prebuilts目录结构不一样，但基本是按平台组织的，找到对应的版本的gdbserver不会花太多时间。

然后在手机上执行命令 run-as package-name gdbserver :port --attach pid

因为android shell 命令是以shell用户执行的，而shell用户id和要attach的进程的用户ID不一样，直接执行gdbserver会报权限错误，所以要使用run-as命令，让gdbserver和要attach的进程的用户ID是一样的。

但这种方法要配置很多东西，Android源码的envsetup.sh提供了一个gdbclient函数，对上面的步骤进行了封装，并做好了配置，这个函数会自动根据机型和版本选择对应的gdb和gdbserver。命令是gdbclient --atttach port pid。在android5.0上要加入run-as命令。

可以用type -a gdbclient查看函数定义

还需执行adb root命令，这样在gdb启动后，可以执行share命令加载所以符号表

nm命令可以读取so文件的符合表，由于C++的类成员函数的命名空间在编译时被被转化成内部字符串，可以用c++filt命令将内部字符串转化成实际的命名空间，例如

nm out/target/product/ferrari/symbols/system/lib64/libwebviewchromium.so | grep -E "[Dd]raw.*[Tt]ext" | c++filt | grep -E "[Dd]raw"

set print pretty on可以格式化结构体显示

使用share filename命令加载包含符号表的so文件

Java调试器和gdb有冲突，要先用Java调试器attach进程，等所有调试信息显示出来后，再用gdb attach进程

gdb -tui会显示一个源代码窗口，方便调试，在TUI模式下可以参考下面的链接进行操作

[https://sourceware.org/gdb/onlinedocs/gdb/TUI.html#TUI](https://sourceware.org/gdb/onlinedocs/gdb/TUI.html#TUI)

Ctrl + x + a可以进入或退出TUI模式

Ctrl + x + o可以切换当前活动的窗口