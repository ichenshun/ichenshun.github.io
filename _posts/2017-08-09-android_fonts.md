---
layout: post
title: 字体处理
categories: [Shell]
keywords: ssh, mysql
---

一个字体文件中一般不会包含所有UNICODE的字形，例如AndroidClock.ttf字体文件只包含了只有数字、A、P、M这三个字母i以及冒号":"这几个字符的字形。但TextView指定使用这个字体文件时，这个字体文件了包含的字符会按这个字体文件的字形显示，而其他字符比如B、C等就会按另外一个字体文件的字形来显示。那么另外一个字体文件是怎么来的呢？答案就是fallback.xml文件定义了在这种找不到字符的字形的情况下，应该从哪些备用的字体文件中找字形。

Skia图形库会读取/system/etc/system_fonts.xml和/system/etc/fallback_fonts.xml中的字体描述，并把system_fonts.xml中的描述的字体作为默认字体，把fallback_fonts.xml中的字体作为备用字体。备用字体的功能如上所述。

在5.0中，使用了一个新的文件来描述这些字体/system/etc/fonts.xml，处理代码是Typeface.init，在通过familyName创建Typeface时，如果fonts.xml中不包含familyName所描述的字体，那么就会使用默认的字体。

