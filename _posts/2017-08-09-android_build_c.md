---
layout: post
title: Android Build
categories: [Android]
keywords: Android, Build
---

C++编译禁用优化选项

在build/core/combo/select.mk中去掉 RELEASE_CFLAGS 变量中的 -O2选项，就可以禁用编译优化了，这样调试时实际代码和源代码是一致的，方便调试

另外在build/core/combo/中的文件是用来配置各个语言编译器选项的

其实，直接在相应模块中的CPPFLAG中添加-O0选项就可以禁用优化了



当通过mm/mmm命名编译指定的模块时，mm/mmm shell函数会设定

ONE_SHOT_MAKEFILE变量，然后调用make编译all_modules，all_modules会依赖ALL_MODULES变量中定义的所有模块。这些模块是在每个模块的makefile中定义的，当在Android.mk中定义LOCAL_PATH，LOCAL_MODULE后，然后include $(BUILD_XXX)类似的文件，最终会include base_rule.mk文件，这个文件中通过ALL_MODULES += my_register_name记录的每个模块中定义的所有module。

直接通过make xxx命令编译系统或者某个模块时，会include所有的模块makefile(即模块目录下的Android.mk文件)，而通过mm/mmm命令编译某个模块时，不会inclulde这些Android.mk，而是只include模块目录下的Android.mk

base_rule.mk中定义了各种文件比如apk、so、ttf的编译步骤

Android.mk是一个模块的makefile，当使用mm/mmm命令build指定的模块时，不会include其他模块的makefile，当使用make命令build系统或者某个模块时，会include所有模块的makefile

PRODUCT_COPY_FILES在buile/core/Makefile文件中处理，最终目标文件被加到ALL_DEFAULT_INSTALLED_MODULES，ALL_DEFAULT_INSTALLED_MODULES中的所有模块被加到INTERNAL_SYSTEMIMAGE_FILES中，然后被加到FULL_SYSTEMIMAGE_DEPS中，最后作为BUILT_SYSTEMIMAGE的依赖模块，BUILT_SYSTEMIMAGE被INSTALLED_SYSTEMIMAGE依赖，INSTALLED_SYSTEMIMAGE被systemimage依赖，所以make systemimage会自动build这些模块

ALL_DEFAULT_INSTALLED_MODULES主要来源于main.mk文件中的modules_to_install和PRODUCT_COPY_FILES以及部分模块的Android.mk文件

字体模块的编译由各个字体目录下的Android.mk来描述的，同时fonts.mk通过PRODUCT_COPY_FILE声明需要复制的system_fonts.xmk、fallback_fonts.xml和fonts.xml，也通过PRODUCT_PACKAGES声明了需要复制的字体文件，但是在Android.mk中也声明的需要复制的字体文件，并通过include $(BUILD_PREBUILT)来复制字体文件

make showcommands xxx可以显示make xxx的命令。原理是build/core/config.mk文件中通过MAKECMDGOALS变量检查传入的make参数中是否包含showcommands，如果包含，那么在build/core/definitions.mk中将hide变量设为空值。因为默认情况下，makefile中的recipe会在开头引用$(hide)变量，当hide变量为空值时，就会显示要执行的命令