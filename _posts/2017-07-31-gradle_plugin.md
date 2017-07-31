---
layout: post
title: 如何编写Gradle插件
categories: [Gradle]
keywords: gradle, plugin
---

[Gradle](https://docs.gradle.org/current/userguide/userguide.html)插件有三种方式

第一种是直接在build.gradle配置文件中编写，第二种是在buildSrc中，第三种是在独立的项目中

那么想给插件添加像android一样的配置选项，应该怎么做呢？

首先定义一个类，类名可以任意写，但一般以Extension或者Option结尾：

```groovy
class SampleExtension {
    def boolean printDebugLog
    def String message
}
```



然后在插件类中的apply方法里面加入如下语句：

```groovy
class SamplePlugin implements Plugin<Project> {
    void apply(Project project) {
        project.extensions.create("sample", SampleExtension)
        ...
        project.task('hello') {
            doLast {
                if (project.sample.printDebugLog) {
                    println "${project.sample.message}"
                }
            }
        } 
    }
}
```

这样就可以在build.gradle中通过sample给插件添加配置了：

```groovy
sample {
    printDebugLog = true
    message = "hello plugin"
}
```

当执行完project.extensions.create("sample", SampleExtension)后，sample就是project的一个属性了，接着就可以通过project.sample访问sample对象。

要注意的是在plugin中引用时，要放在task的action里面或着其他回调方法里面，如果紧接着create语句访问sample对象，是取不到值的。因为apply方法被调用时sample对象还没被初始化。