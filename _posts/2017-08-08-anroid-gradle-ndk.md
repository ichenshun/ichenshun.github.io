---
layout: post
title: Android Gradle NDK
categories: [Android]
keywords: Android, NDK, Gradle
---

Gradle NDK  编译有四种方式

第一种是 Gradle Plugin稳定版支持的方式，直接在gradle 配置文件中写ndk配置就可以了，大致方式如下，gradle插件会自动生成Android.mk并调用ndk-build编译

```groovy
android {
     defaultConfig {
          ndk {
            abiFilter "x86"
            moduleName "hello-jni"
            stl "gnustl_static"
            cFlags "-I/opt/local/include/opencv -I/opt/local/include"
            ldLibs libsDir + "x86/libopencv_core.a"
            ldLibs libsDir + "x86/libopencv_ts.a"
            ldLibs libsDir + "x86/libopencv_contrib.a"
            ldLibs libsDir + "x86/libopencv_ml.a"
            ldLibs libsDir + "x86/libopencv_java.so"
            ldLibs "log"
            ldLibs "z", "jnigraphics"
          }
     }

     sourceSets {
          main {
               jni.srcFiles ['src/main/jni']
          }
     }
}
```



第二种方式是在Gradle配置文件种指定外部ndk-build，并作简单的打包配置，大致方式如下，同时还需要提供Android.mk文件，gradle根据配置调用ndk-build

```groovy
android {
     externalNativeBuild {
         ndkBuild {
            path 'Android.mk'
         }
     }
     defaultConfig {
          externalNativeBuild {
              ndkBuild {
                targets "target1", "target2"
                arguments "NDK_APPLICATION_MK:=Application.mk"
                cFlags "-DTEST_CFLAG1", "-DTEST_CFLAG2"
                cppFlags "-DTEST_CPP_FLAG2", "-DTEST_CPP_FLAG2"
                abiFilters "armeabi-v7a", "armeabi"
         } 
       }
     }
}
```

第三种是是在Gradle种配置CMake，和第二种方式类似，把ndkBuild改成CMake就可以了

第四种方式是使用实验版的Gradle插件，直接在Gradle配置中写ndk build，并提供了丰富的dsl，可以参考官方网站[http://tools.android.com/tech-docs/new-build-system/gradle-experimental#TOC-Ndk-Integration](http://tools.android.com/tech-docs/new-build-system/gradle-experimental#TOC-Ndk-Integration)

另，在ndk-build时，如果制定了目标文件，那么生成的目标文件是原始的，没有经过任何裁剪的，生成的so文件会比较大，如果指定目标文件，会自动裁剪。上面的第一种方式会自动裁剪，第二种方式不会，但是在生成APP时会执行裁剪的过程，生成aar是不会裁剪

遇到的问题：

1. 如果出现 Execution failed for task':app:transformNative_libsWithStripDebugSymbolForDebug
     java.lang.NullPointerException (no error message)
   1. 可能原因是so包含的arm64和x86平台版本，需要将compileSdkVersion设21以上