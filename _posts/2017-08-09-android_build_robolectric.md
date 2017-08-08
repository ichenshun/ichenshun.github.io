---
layout: post
title: 自定义编译环境中使用Robolectric
categories: [Android]
keywords: Android, Build, Robolectric
---

下载源码后，首先执行scripts目录下的install-robolectric.sh，这个脚本会将Android Sdk中的support包、maps包等安装到~/.m2/仓库中，如果提示版本不兼容，那么改一下出错项目的pom.xml配置，并且改一下安装脚本，然后执行mvn package打包robolectric。

使用下面的命令运行测试用例，要用org.junit.runner.JUnitCore 去跑测试用例，并且在测试Class中指定如下标签

```
@RunWith(org.robolectric.RobolectricTestRunner.class)

```

@Config(manifest = "packages/apps/MiuiSystemSdk/library/AndroidManifest.xml")

另外这些jar必须放在classpath中，如果解包后由于缺少shadows-core-3.0.jar中的MATEINFO/services文件，会出现找不到ShadowAdapter的错误

java -cp out/target/common/obj/APPS/MiuiSystemSdkTest_intermediates/classes:/home/chenshun/lib/android-sdk-linux/platforms/android-19/android.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/robolectric-3.0.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/robolectric-annotations-3.0.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/robolectric-resources-3.0.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/robolectric-utils-3.0.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/shadows-core-3.0.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/ant-1.8.0.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/asm-debug-all-5.0.2.jar:packages/apps/MiuiSdk/samples/MiuiDemo/tests/libs/bcprov-jdk16-1.46.jar:/home/chenshun/.m2/repository/com/almworks/sqlite4java/sqlite4java/0.282/sqlite4java-0.282.jar:out/target/common/obj/APPS/miuisystem_intermediates/classes.jar:out/target/common/obj/APPS/miui_intermediates/classes.jar:out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes.jar org.junit.runner.JUnitCore  miui.util.HardwareInfoTest