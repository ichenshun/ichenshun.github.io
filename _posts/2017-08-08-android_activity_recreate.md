---
layout: post
title: BackupManagerService备份APK的特点
categories: [Android]
keywords: Android, Library, Gradle
---

备份Apk时会写下installerName，源码在writeAppManifest方法中

还原时会使用备份的installerName安装Apk，源码在restoreOneFile