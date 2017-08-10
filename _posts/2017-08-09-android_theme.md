---
layout: post
title: Android主题和样式
categories: [Android]
keywords: Android, Theme
---

```xml
<style name="Theme"/>
<style name="Theme.NoTitleBar"/>
```

编译后对应类似0x010300f0的值，资源类型是style，R定义是R.style.Theme_NoTitleBar

```xml
<TextView style="@style/XXXX"></TextView>
```

这样直接通过style引用的值，在inflate时会自动展开，因为TextView的构造方法中没有解析style属性的代码。如果在编译时展开，那么就会导致无法根据设备配置自动选择合适的资源。

```xml
<style name="Widget.Button">
    <item name="textAppearance">?attr/textAppearanceSmall</item>
</style>
```

直接引用主题定义的textAppearanceSmall属性的值

Theme定义

Theme

Theme.Light就是默认Theme

Theme.Dark也继承自默认Theme，但定义了不同的样式，同时也继承自android:Theme.Holo

Theme.Dark.RemoteViews

Theme.Dark parent android:Theme.Holo

textAppearance是系统定义的一个属性，在多个styleable中被包含：Theme、TextViewAppearance、FastScroll、TextView。

TextView在构造时，先读取TextViewAppearance styleable中定义的属性，TextViewAppearance styleable中只有一个textAppearance属性，然后读取textAppearance属性的值，这个属性值是一个style的引用，然后用TextAppearance styleable去过滤这个style中的属性，只留下textColor、textSize等文字字体样式相关的属性。

这说明一般属性的值不会自动展开，上面的textAppearance的属性值没有自动展开，而需要先读取这个属性的值，然后再从这个值中读取需要的属性

另外android.R.theme中没有直接指定textStyle这样具体的文字样式属性值，而是定义了textAppearance属性的值，这个值是一个style的引用，通过这个style来定义具体的文字样式。

综合上面两点，在代码中通过Context.getTheme.resolveAttribute(android.R.attr.textStyle)获取不到值。

textAppearance有几个名字一样但类型不一样的定义

1. textAppearance，attr类型，取值是引用类型，一般引用style类型的资源
2. TextAppearance，styleable类型，收集了textColor、textSize等字体样式相关的属性集合，用于TextView解析字体样式属性
3. TextAppearance，style类型，定义了基本的文字样式，给TextView定义文字样式用   