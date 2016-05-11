---
layout:     post
title:      "Android 绘图机制"
subtitle:   ""
date:       2016-05-11
author:     "WunWun"
header-img: "img/in-post/android-heros/android-deal-picture.jpg"
tags:
    - Android
    - 绘图
---

## PorterDuffXfermode

正常的情况下，在已有的图像上绘制新的图像，新图像将遮挡原来的图像。PorterDuffXfermode设置的是两个图层叠置时的显示方式。

PorterDuffXfermode共有16种模式。canvas原有的图像可以理解为背景，就是dst；新画上去的图像可以理解为前景，就是src。

![java-javascript](/img/in-post/android-heros/android-PorterDuffXfermode.jpg)
<small class="img-hint">PorterDuffXfermode</small>

PorterDuffXfermode使用非常简单。

	Canvas canvas = new Canvas(dstBitmap);  
	paint.setXfermode(new PorterDuffXfermode(Mode.SRC_IN));    
	canvas.drawBitmap(srcBitmap, 0f, 0f, paint); 

## Shader

Shader又被称之为着色器，渲染器，用来实现一系列的渐变，渲染效果。主要包括以下几种

- BitmapShader  位图Shader

- LinearShader  线性Shader

- RadialShader  光束Shader

- SweepGradient  梯度Shader

- ComposeShader  混合Shader

## PathEffect

PathEffect是指，用各种笔触效果来绘制一个路径。

- CornerPathEffect

- DiscretePathEffect

- DashPathEffect

- PathDashPathEffect

- ComposePathEffect

使用方法如下

	mPath = new Path();
    mPath.moveTo(0, 0);
    for (int i = 0; i <= 30; i++) {
        mPath.lineTo(i * 35, (float) (Math.random() * 100));
    }
    mEffects = new CornerPathEffect(30);
    mPaint.setPathEffect(mEffects);
    canvas.drawPath(mPath, mPaint);

PathEffect的各种效果

![java-javascript](/img/in-post/android-heros/android-path-effect.jpg)
<small class="img-hint">PathEffect的各种效果</small>