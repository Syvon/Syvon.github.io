---
layout:     post
title:      "Android中的context"
subtitle:   ""
date:       2016-05-03
author:     "WunWun"
header-img: "img/in-post/android-heros/android-context.png"
tags:
    - Android
    - context
---

    Android中的Context，可以理解为当前对象在程序中所处的一个环境，一个与系统交互的过程。
    Activity，Service，Application都继承自Context。
    Android应用程序会在如下所示的几个时间点创建应用上下文Context。
    * 创建Application
    * 创建Activity
    * 创建Service
    创建Context的时机就是在创建Context的实现类的时候。当应用程序第一次启动时，Android系统都会创建一个Application对象，同时创建Application Context，所有的组件都共同拥有这样一个Context对象，**这个应用上下文对象贯穿整个应用进程的生命周期，为应用全局提供了功能和环境支持**。
    创建Activity和Service组件时，系统也会给它们提供运行的上下文环境，即创建Activity实例，Service实例的Context对象。所以在Activity中获取Context对象时，可以直接使用this，而在匿名内部类中，就必须指定XXXActivity.this再能获得该Activity的Context对象。
    当然，你也可以通过getApplicationContext方法来获取整个APP的Context，但是这种方法获得的是整个应用的上下文引用，这与某个组建的上下文引用，在某些时候还是有区别的。












