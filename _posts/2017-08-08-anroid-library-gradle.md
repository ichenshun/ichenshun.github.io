---
layout: post
title: Android Library Project发布jar
categories: [Android]
keywords: Android, Library, Gradle
---

Android Library Project只能发布aar，加入下面的配置后，可以同时发布jar

```groovy
task makeJar(type: Jar) {
    from('build/intermediates/classes/release/')
    include('*/.class')
    exclude('**/BuildConfig.class')
}

artifacts {
    archives makeJar
}
```

