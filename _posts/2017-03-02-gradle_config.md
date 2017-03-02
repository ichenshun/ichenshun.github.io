---
layout: post
title: Gradle属性配置
categories: Gradle
keywords: Gradle, 配置
---

在Gradle配置文件中中可以通过如下方式访问Project的属性：

```groovy
project.hasProperty('key')
project.property('key')
```

这些属性怎么定义呢？有几种方式

第一种是在project的根目录的gradle.properties文件中定义属性

```groovy
# <project_root>/gradle.properties
key=value
```

这个属性文件一般会check in到代码仓库中

第二种是在~/.gradle/gradle.properties文件中定义属性

```groovy
# ~/.gradle/gradle.properties
key=value
```

这个属性文件不被check in到代码仓库，属于本地定义