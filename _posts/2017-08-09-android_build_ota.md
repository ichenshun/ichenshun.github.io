---
layout: post
title: 编译OTA包
categories: [Android]
keywords: Android, Build, OTA
---

首先调用make target-files-package编译ROM的所有文件

然后在out/target/product/xxxx/目录下逐个将ROM中的文件刷入到手机中，可以参考对应机型的flash脚本，不同机型的ROM文件包含的内容不一样，刷机方式也不一样

然后将out/target/product/xxxx/obj/PACKAGING/target_files_intermediates/xxxxx-target_files-eng.xxxx.zip

拷贝并且更名放到目录~/OTA/xxxx-target_files-eng.A.zip

1. 在代码中产生一些更新
2. 第二次调用make  target-files-package或者make otapackage，将编译生成的out/target/product/xxxx/obj/PACKAGING/target_files_intermediates/xxxx-target_files-eng.xxxx.zip 拷贝并且更名放到目录/OTA/xxxx-target_files-eng.B.zip
3. 在src根目录下执行./build/tools/releasetools/ota_from_target_files -i <A包> <B包> <差分包名>。这里必须在src根目录下执行,因为ota_from_target_files.py这个脚本里面写定了相对路径的引用文件。

如：./build/tools/releasetools/ota_from_target_files -v -t MMC -i ~/OTA/xxxx-target_files-eng.A.zip ~/OTA/xxxx-target_files-eng.B.zip ~/OTA/update.zip  

~/OTA/update.zip  就是升级用的差分包。

注意：-t MMC 是指使用文件格式为ext4,默认为mtd,即yaffs2。因为我们这个系统使用了ext4文件系统的支持。具体的内容可以看分区表文件src/

具体的参数含义为 -v显示具体命令，-i 为产生增量包。