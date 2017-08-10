---
layout: post
title: IntelliJ Java class path & module source
categories: [Idea]
keywords: Idea
---

sdk class path指定了Project依赖的类的声明，这些类由实际的运行环境提供，在IntelliJ中这些类只是参与编译，并且为编辑器提供语法提示和自动完成功能，module source表示Project中包含的源代码，这些源代码由我们自己创建。编辑器同样也需要依赖module source提供语法提示和自动完成功能。

在IntelliJ的Project Structure->Modules->Dependicies中，列出了一个Project依赖的所有类来源，Module Source也作为依赖来源列在其中，编辑器按找列表中的顺序提供语法提示和自动完成功能。举个例子，假如sdk class path定义了一个android.content.res.AssetManager类，在Module Source中用户自己也定义了一个android.content.res.AssetManager类，假如sdk class path是类来源列表的第一项的话，编辑器会直接读取sdk class path中的AssetManager进行语法提示和自动完成，当在用户自定义的AssetManager中假如一个publi static int test变量时，在其他地方输入AssetManager.te时，不会自动提示补全test变量，因为sdk class path中的AssetManager类没有定义test变量。

   