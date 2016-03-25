---
layout:     post
title:      "Android简介"
subtitle:   "《第一行代码》复习笔记1"
date:       2016-03-24
author:     "WunWun"
header-img: "img/android-music.jpg"
tags:
    - Android
    - 学习笔记
---

> 这是《第一行代码》复习笔记的第一章.


## Android系统架构

1. Linux内核层
	Android系统基于Linux2.6内核，这一层为Android设备的各种硬件提供了底层的驱动，如显示驱动，音频驱动，照相机驱动，蓝牙驱动等。
2. 系统运行库层
	这一层通过一些C/C++库来为Android系统提供了主要的特性支持。如SQLite库提供了数据库的支持，OpenGL/ES库提供了3D绘图的支持，Webkit库提供了浏览器内核的支持。
	这一层还有Android运行时库，它提供了一些核心库和Dalvik虚拟机。核心库能够允许开发者使用Java语言来编写Android应用；Dalvik虚拟机使得每一个Android应用都能运行在独立的进程当中，并且拥有一个自己的Dalvik虚拟机实例。
3. 应用框架层
	这一层主要提供了构建应用程序时可能用到的各种API，Android自带的一些核心应用就是使用这些API完成的。
4. 应用层
	所有安装在手机上的应用程序都是属于这一层的。

![Android-System-Architecture](/img/in-post/Android-System-Architecture.png)
<small class="img-hint">Android系统架构图</small>


## 项目的目录结构

1. src
	src是放置所有Java代码的地方
2. gen
	这个目录里的内容都是自动生成的，主要有一个R.java的文件，你在项目中添加的任何资源都会在其中生成一个相应的资源id。这个文件永远不要去手动修改它。
3. assets
	这个目录用的不多，主要可以存放一些随程序打包的文件，在程序运行时可以动态地读取到这些文件的内容。
4. bin
	主要包含了一些在编译时自动产生的文件。
5. libs
	如果项目中用到了第三方Jar包，就需要把这些Jar包都放在libs目录下。这个目录下的Jar包都会自动添加到构建路径中。
6. res
	项目中使用到的所有图片，布局，字符串等资源都要存放在这个目录下。R.java中的内容也是根据这个目录下的文件自动生成的。
7. AndroidManifest.xml
	整个Android项目的配置文件，在程序中定义的所有四大组件都需要在这个文件里注册。
8. project.properties
	通过一行代码指定了编译程序时所使用的SDK版本。


## 其他

1. SDK选项
	- Minimum Required SDK 是指程序最低兼容的版本。
	- Target SDK 是指你在该目标版本上已经做过了成分的测试，系统不会再帮你在这个版本上做向前兼容的操作。
	- Compile With 是指程序将使用哪个版本的SDK进行编译。
2. 如何引用字符串等资源。
	- 在代码中通过R.string.hello_world可以获得该字符串的引用。
	- 在XML中通过@string/hello_world可以获得该字符串的引用。

> 其中string部分是可以替换的，如果引用图片资源就可以替换成drawable，如果引用布局文件可以替换成layout

