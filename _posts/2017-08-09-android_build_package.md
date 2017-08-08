---
layout: post
title: 重定义BUILD_PACKAGE的APK安装路径
categories: [Android]
keywords: Android, Build
---

install_path := $(TARGET_OUT_DATA)/miui/xxxxl

LOCAL_POST_INSTALL_CMD := mkdir -p $(install_path); \

mv $(TARGET_OUT_APPS)/$(LOCAL_PACKAGE_NAME)/$(LOCAL_PACKAGE_NAME).apk $(TARGET_OUT_DATA)/miui/xxxxl/xx.apk; \

rm $(TARGET_OUT_APPS)/$(LOCAL_PACKAGE_NAME) -r