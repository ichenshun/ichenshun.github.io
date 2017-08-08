---
layout: post
title: Android AAPT分析和使用（草稿）
categories: [Android]
keywords: Android, AAPT
---

xmlns声明

xml文件中的xmlns声明格式是

```xml
xmlns:android="http://schemas.android.com/apk/res/android"
```

如果是使用自定义属性，声明格式是

```xml
xmlns:app="http://schemas.android.com/apk/res/app.package.name"
```

aapt编译时，会解析包名，比如android，app.package.name，然后拿这个包名到-I参数指定的base package中找对应的定义。如果将android声明替换成这种

```xml
xmlns:android="http://schemas.android.com/apk/res/afdafa"
```

那么编译时会提示下面的错误

No resource identifier found for attribute 'versionCode' in package 'afdafda'

也就是说包名必须是实实在在存在的包名，不能用任意的名字

--auto-add-overlay

指定多个res目录时，aapt就会认为有资源覆盖，在这种情况下，需要所有定义的资源在base package中有定义，如果没有定义，就会提示下面的错误

Resource at fade_duration appears in overlay but not in the base package; use to add.

指定这个参数后，就相当于自动给每个资源添加一个覆盖（？？？资源覆盖是什么意思，为什么需要覆盖）

--feature-of/--feature-after

feature package is a split apk that is feature of base package，指定--feature-of base.apk，表示当前编译的包是base.apk的一个feature包，feature包可以引用base.apk中的资源

实际测试编出来的包资源ID并没有和base.apk错开，通过代码分析和调试，在获取初始type id时，并不是拿base.apk的最大type id，而是尝试获取自身包的最大type id，但此时自身包还没编译出来，获取的id始终为0

-c

指定一个配置列表，当一个资源有多个配置时，保留-c指定的配置，其他的配置会被删掉，但如果只有一个配置，那么还是会保留



资源文件，但目录结构和源码中的资源目录结构不一样。源码中的资源目录结构如下：

res

|__drawable

​     |__icon.png

|__drawable-hdpi

​     |__icon.png

|__layout

​     |__main.xml

AaptAsset的目录结构如下：

res (AaptDir)

|__drawable (AaptDir)

​     |__icon.png (AaptGroup)

​           |__default,hdpi(AaptConfigList)

​                |__res/drawable/icon.png (AaptFile(AaptGroupEntry))

​                |__res/drawable-hdpi/icon.png (AaptFile(AaptGroupEntry))

|__layout (AaptDir)

​     |__main.xml (AaptGroup)

​          |__default(AaptConfigList)

​               |__res/layout/main.xml (AaptFile(AaptGroupEntry))

AaptGroupEntry其实就是一个资源的config描述，是对ConfigDescription的分装，添加了用于比较运算的操作。



代码和资源分开

资源编译器aapt如何使用，各个参数各有什么用处

有哪些资源，分别用于什么目的，各种资源如何编译

代码中如何访问资源

第一部分：打包资源

​    第一节：资源打包过程

​          最简单的编译方法是执行下面的编译命令

​     aapt package -M AndroidManifest.xml -S res -I framework-res.apk -F package.apk

​           package 表示打包资源

​     -M 指定App的配置文件一般以AndroidManifest.xml命名

​     -S 指定资源目录

​     -I 指定需要引用的资源包

​     -F 指定

​    第二节：原始资源类型

​    第三节：编译后的资源结构

​    第四节：各个资源编译过程

第二部分：代码中访问资源