---
layout: post
title: Gradle使用SNAPSHOT版本的依赖模块问题
categories: [Gradle]
keywords: gradle, snapshot
---

参考链接：[https://docs.gradle.org/current/userguide/dependency_management.html#sub:dynamic_versions_and_changing_modules](https://docs.gradle.org/current/userguide/dependency_management.html#sub:dynamic_versions_and_changing_modules)

参考链接：[https://docs.gradle.org/current/userguide/dependency_management.html#sec:cache_command_line_options](https://docs.gradle.org/current/userguide/dependency_management.html#sec:cache_command_line_options)

 

Gradle支持动态版本和持续更新的模块，有时候需要自动使用模块的最新版本，Maven的SNAPSHOT模块就是持续更新的模块，在Gradle中使用这样的模块时，Gradle会自动下载最新的版本。但是Gradle对此有缓存，默认缓存24小时，也就下载最新版本的模块后，如果24小时内模块又有更新，Gradle不会立即下载最新的版本，而是在24小时后再去下载更新。

这会导致对模块的更新不会立即反应到使用端，解决这一问题的版本是在命令行中使用--refresh-dependencies参数，强制下载最新版本。

gradle clean build --refresh-dependencies