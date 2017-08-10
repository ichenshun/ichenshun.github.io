---
layout: post
title: Android资源覆盖
categories: [Android]
keywords: Android, Resource
---

在5.0上进行资源覆盖，是在属性级别进行覆盖，而不是style级别。

比如framework里定义了如下style

```xml
<style name="TextAppearance.StatusBar.EventContent.Title">
    <item name="textSize">30sp</item>
    <item name="textColor">#ffcccccc</item>
    <itme name="textStyle">bold</item>
</style>
```

在overlay里也定义同名的style，但属性不一样

```xml
<style name="TextAppearance.StatusBar.EventContent.Title">
    <item name="textColor">#ffffffff</item>
</style>
```

覆盖的结果是textColor变成了#fffffffff，而textSize和textStyle仍然保持30sp和bold

在实现上是由aapt资源编译工具通过参数来支持的。

   