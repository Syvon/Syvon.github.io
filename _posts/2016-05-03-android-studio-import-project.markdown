---
layout:     post
title:      "如何正确的导入Android Studio工程"
subtitle:   ""
date:       2016-05-03
author:     "WunWun"
header-img: "img/in-post/android-heros/android-studio-import-project.jpg"
tags:
    - Android Studio
    - Android
    - problem
---

由于Android Studio中的Gradle经常会有版本更新。如果某个项目使用的是Gradle2.0进行的编译，而本地又没有该版本的Gradle的时候，Android Studio就回去下载这个版本的Gradle。在天朝，呵呵，导致Android Studio卡死。

### 正确的导入方法
首先在本地用当前版本的Gradle创建一个正常的项目，保证可以编译通过即可。然后，用本地项目中的**Gradle**文件夹和**build.gradle**文件，去替换要导入项目中这两个文件












