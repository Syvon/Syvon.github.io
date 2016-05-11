---
layout:     post
title:      "Android Scroll分析1"
subtitle:   "坐标系，MotionEvent"
date:       2016-05-08
author:     "WunWun"
header-img: "img/in-post/android-heros/android-scroll.jpg"
tags:
    - Android
    - 滑动
---

### Android坐标系

在Android中，将屏幕最左上角的顶点作为Android坐标系的原点，从这个点向右是X轴正方向，从这个点向下是Y轴正方向。


系统提供了getLocationOnScreen(intLocation[])这样的方法来获取Android坐标系中点的位置，即该视图左上角在Android坐标系中的坐标。

在触控事件中使用getRawX(),getRawY()方法所获得的坐标同样是Android坐标系中的坐标。

### 视图坐标系

视图坐标系，描述了子视图在父视图中的位置关系。视图坐标系同样是以原点向右为X轴正方向，以原点向下为Y轴正方向，以父视图左上角为坐标原点。



在触控事件中，通过getX(),getY()所获得的坐标就是视图坐标系中的坐标。

### 触控事件

MotionEvent封装了一些常用的事件常量，它定义了触控事件的不同类型。

    public static final int ACTION_DOWN          = 0;  //单点触摸按下动作

    public static final int ACTION_UP            = 1;  //单点触摸离开动作

    public static final int ACTION_MOVE          = 2;  //触摸点移动动作

    public static final int ACTION_CANCEL        = 3;  //触摸动作取消

    public static final int ACTION_OUTSIDE       = 4;  //触摸动作超出边界

    public static final int ACTION_POINTER_DOWN  = 5;  //多点触摸按下动作

    public static final int ACTION_POINTER_UP    = 6;  //多点离开动作

通常会在onTouchEvent(MotionEvent event)方法中通过event.getAction()方法来获取事件的类型，并使用switch-case方法来进行筛选。

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:

                break;
            case MotionEvent.ACTION_MOVE:

                break;
            case MotionEvent.ACTION_UP:
                // 手指离开时，执行滑动过程

                invalidate();
                break;
        }
        return true;
    }

### 获取坐标值的各种方法



- View提供的获取坐标方法

getTop():获取到的是View自身的顶边到其父布局顶边的距离

getLeft():获取到的是View自身的左边到其父布局左边的距离

getRight():获取到的是View自身的右边到其父布局左边的距离

getBottom():获取到的是View自身的底边到其父布局顶边的距离

- MotionEvent提供的获取坐标方法

getX():获取点击事件距离控件左边的距离，即视图坐标。

getY():获取点击事件距离控件顶边的距离，即视图坐标。

getRawX():获取点击事件距离整个屏幕左边的距离，即绝对坐标。

getRawY():获取点击事件距离整个屏幕顶边的距离，即绝对坐标。