---
layout:     post
title:      "Android 图像处理之色彩特效处理"
subtitle:   ""
date:       2016-05-11
author:     "WunWun"
header-img: "img/in-post/android-heros/android-deal-picture.jpg"
tags:
    - Android
    - ColorMatrix
---

## 颜色矩阵分析

色彩处理中，从三个角度来描述一个图像

- 色相（H），  色彩的基本属性，就是平常所说的颜色名称，如红色、黄色等。

- 饱和度（S），色彩的纯度，越高色彩越纯，低则逐渐变灰，取0-100%的数值。

- 明度（V），  亮度（L），取0-100%。

在Android中使用一个颜色矩阵——ColorMatrix，来处理图像的色彩效果。ColorMatrix是一个4*5的数字矩阵，它用来对图片的色彩进行处理。而像素点，都有一个颜色分量矩阵用来保存颜色的RGBA值。

![java-javascript](/img/in-post/android-heros/android-color-matrix.png)
<small class="img-hint">ColorMatrix</small>

![java-javascript](/img/in-post/android-heros/android-color-rgba.png)
<small class="img-hint">颜色分量矩阵</small>

要想改变一张图片的颜色效果，只需要改变图像的颜色分量矩阵即可。通过颜色矩阵可以很方便的修改图像的颜色分量矩阵。假设修改后的图像颜色分量矩阵为C1，则有如下所示的颜色分量矩阵计算公式。

![java-javascript](/img/in-post/android-heros/android-color-matrix-deal.png)
<small class="img-hint">颜色分量矩阵计算公式</small>

从这个公式可以发现，
- 第一行参数abcde决定了图像的红色成分，
- 第二行参数fghij决定了图像的绿色成分，
- 第三行参数klmno决定了图像的蓝色成分，
- 第四行参数pqrst决定了图像的透明度，
- 第五列参数ejot是颜色的偏移量。

## 应用ColorMatrix改变色光属性

在颜色矩阵中，封装了一些API来快速调整这些色相，饱和度，亮度，而不用每次都去计算这些矩阵的值。

- setRotate(int axis, float degrees)

第一个参数，使用0,1，2来代表Red，Green，Blue三种颜色的处理；第二个参数，就是需要处理的值。

	ColorMatrix hueMatrix = new ColorMatrix();
    hueMatrix.setRotate(0, hue);
    hueMatrix.setRotate(1, hue);
    hueMatrix.setRotate(2, hue);

- setSaturation(float sat)

设置颜色的饱和度

	ColorMatrix saturationMatrix = new ColorMatrix();
    saturationMatrix.setSaturation(saturation);

- setScale(float rScale, float gScale, float bScale, float aScale)

设置颜色的亮度

    ColorMatrix lumMatrix = new ColorMatrix();
    lumMatrix.setScale(lum, lum, lum, 1);

- postConcat(ColorMatrix postmatrix)

将矩阵的作用效果混合

    ColorMatrix imageMatrix = new ColorMatrix();
    imageMatrix.postConcat(hueMatrix);
    imageMatrix.postConcat(saturationMatrix);
    imageMatrix.postConcat(lumMatrix);

## 常用图像颜色矩阵处理效果

将对应的颜色矩阵值作用到原图像上，从而形成新的色彩风格的图像。

- 灰度效果

		0.33f,0.59f,0.11f,0, 0,
		0.33f,0.59f,0.11f,0, 0,
		0.33f,0.59f,0.11f,0, 0,
		0,    0,    0,    1, 0,

- 图像反转

		-1, 0, 0, 1, 1,
		 0,-1, 0, 1, 1,
		 0, 0,-1, 1, 1,
		 0, 0, 0, 1, 0,

- 怀旧效果

		0.393f,0.769f,0.189f,0, 0,
		0.349f,0.686f.0.168f,0, 0,
		0.242f,0.534f,0.131f,0, 0,
		0,     0,     0,     0, 0,

- 去色效果

		1.5f, 1.5f, 1.5f, 0, -1,
		1.5f, 1.5f, 1.5f, 0, -1,
		1.5f, 1.5f, 1.5f, 0, -1,
		0,    0,    0,    1,  0,

## 像素点分析

作为更精确的图像处理方式，可以通过改变每个像素点的具体ARGB值，来达到处理一张图像效果的目的。

提取整个bitmap中的像素点

	bm.getPixels(oldPx, 0, width, 0, 0, width, height);

提取每个像素具体的ARGB值

	color = oldPx[i];
    r = Color.red(color);
    g = Color.green(color);
    b = Color.blue(color);
    a = Color.alpha(color);

当获取到具体的颜色值后，就可以通过相应的算法来修改它的ARGB值，生成一张新的图像。

    r = (r - r1 + 127);
    g = (g - g1 + 127);
    b = (b - b1 + 127);

将新的RGBA值合成像素点

	newPx[i] = Color.argb(a, r, g, b);
	
将处理之后的像素点数组重新设置给Bitmap

	bmp.setPixels(newPx, 0, width, 0, 0, width, height);