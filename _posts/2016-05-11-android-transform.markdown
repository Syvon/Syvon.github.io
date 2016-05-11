---
layout:     post
title:      "Android图像处理之图形特效处理"
subtitle:   ""
date:       2016-05-11
author:     "WunWun"
header-img: "img/in-post/android-heros/android-deal-picture.jpg"
tags:
    - Android
    - 图形特效
---

## Android图形变换矩阵

对于图像的图形变换，Android通过矩阵来进行处理，每个像素点表达了其坐标的X,Y信息。Android的图形变换矩阵是一个3*3的矩阵。

![java-javascript](/img/in-post/android-heros/android-transform-matrix.png)
<small class="img-hint">图形变换矩阵</small>

- 平移变换

![java-javascript](/img/in-post/android-heros/android-translate.jpg)
<small class="img-hint">平移变换</small>

- 旋转变换

![java-javascript](/img/in-post/android-heros/android-rotate.jpg)
<small class="img-hint">旋转变换</small>

如果以任意点O为旋转中心来进行旋转变换。

1. 将坐标原点平移到O点

2. 以坐标原点为中心进行旋转变换

3. 将坐标原点还原

- 缩放变换

![java-javascript](/img/in-post/android-heros/android-scale.jpg)
<small class="img-hint">缩放变换</small>

- 错切变换

![java-javascript](/img/in-post/android-heros/android-skew.jpg)
<small class="img-hint">错切变换</small>

## 像素块分析

> Draw the bitmap through the mesh, where mesh vertices are evenly distributed across the bitmap. There are meshWidth+1 vertices across, and meshHeight+1 vertices down. The verts array is accessed in row-major order, so that the first meshWidth+1 vertices are distributed across the top of the bitmap from left to right. 

drawBitmapMesh()将图像分成了一个个的小块，然后通过改变每一个图像小块来修改整个图像。

实例：旗帜飘动的效果

	public class FlagBitmapMeshView extends View {	

	    private final int WIDTH = 200;
	    private final int HEIGHT = 200;
	    private int COUNT = (WIDTH + 1) * (HEIGHT + 1);
	    private float[] verts = new float[COUNT * 2];
	    private float[] orig = new float[COUNT * 2];
	    private Bitmap bitmap;
	    private float A;
	    private float k = 1;	

	    public FlagBitmapMeshView(Context context) {
	        super(context);
	        initView(context);
	    }	

	    public FlagBitmapMeshView(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        initView(context);
	    }	

	    public FlagBitmapMeshView(Context context, AttributeSet attrs,
	                              int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        initView(context);
	    }	

	    private void initView(Context context) {
	        setFocusable(true);
	        bitmap = BitmapFactory.decodeResource(context.getResources(),
	                R.drawable.test);
	        float bitmapWidth = bitmap.getWidth();
	        float bitmapHeight = bitmap.getHeight();
	        int index = 0;
	        for (int y = 0; y <= HEIGHT; y++) {
	            float fy = bitmapHeight * y / HEIGHT;
	            for (int x = 0; x <= WIDTH; x++) {
	                float fx = bitmapWidth * x / WIDTH;
	                orig[index * 2 + 0] = verts[index * 2 + 0] = fx;
	                orig[index * 2 + 1] = verts[index * 2 + 1] = fy + 100;
	                index += 1;
	            }
	        }
	        A = 50;
	    }	

	    @Override
	    protected void onDraw(Canvas canvas) {
	        flagWave();
	        k += 0.1F;
	        canvas.drawBitmapMesh(bitmap, WIDTH, HEIGHT,
	                verts, 0, null, 0, null);
	        invalidate();
	    }	

	    private void flagWave() {
	        for (int j = 0; j <= HEIGHT; j++) {
	            for (int i = 0; i <= WIDTH; i++) {
	                verts[(j * (WIDTH + 1) + i) * 2 + 0] += 0;
	                float offsetY =
	                        (float) Math.sin((float) i / WIDTH * 2 * Math.PI +
	                                Math.PI * k);
	                verts[(j * (WIDTH + 1) + i) * 2 + 1] =
	                        orig[(j * WIDTH + i) * 2 + 1] + offsetY * A;
	            }
	        }
	    }
	}