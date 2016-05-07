---
layout:     post
title:      "创建自定义控件1"
subtitle:   "对现有控件进行扩展"
date:       2016-05-04
author:     "WunWun"
header-img: "img/in-post/android-heros/android-hello.jpg"
tags:
    - Android
    - 自定义控件
---

## 实现自定义控件的方法

- 对现有控件进行扩展

- 通过组合来实现新的控件

- 重写View来实现全新的控件

在View中比较重要的回调方法

- onFinishInflate() 从XML加载组件后回调

- onSizeChanged() 组件大小改变时回调

- onMeasure() 回调该方法来进行测量

- onLayout() 回调该方法确定显示的位置

- onTouchEvent() 监听到触摸事件时回调

## 对现有控件进行扩展

一般来说，我们可以在onDraw()方法中对原生控件行为进行扩展。

程序调用super.onDraw(canvas)方法来实现原生控件的功能，但是在调用super.onDraw()方法之前（绘制文字前）和之后（绘制文字后），我们都可以实现自己的逻辑，完成自己的操作。

	@Override
    protected void onDraw(Canvas canvas) {
        //绘制文本内容前
        super.onDraw(canvas);
        //绘制文本内容后
    }

1. 绘制TextView的背景

	@Override
    protected void onDraw(Canvas canvas) {
        // 绘制外层矩形
        canvas.drawRect(
                0,
                0,
                getMeasuredWidth(),
                getMeasuredHeight(),
                mPaint1);
        // 绘制内层矩形
        canvas.drawRect(
                10,
                10,
                getMeasuredWidth() - 10,
                getMeasuredHeight() - 10,
                mPaint2);
        canvas.save();
        // 绘制文字前平移10像素
        canvas.translate(10, 0);
        // 父类完成的方法，即绘制文本
        super.onDraw(canvas);
        canvas.restore();
    }

2. 实现动态文字的闪动效果

	@Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (mViewWidth == 0) {
            mViewWidth = getMeasuredWidth();
            if (mViewWidth > 0) {
                mPaint = getPaint();
                mLinearGradient = new LinearGradient(
                        0,
                        0,
                        mViewWidth,
                        0,
                        new int[]{
                                Color.YELLOW, 0xffffffff,
                                Color.BLUE},
                        null,
                        Shader.TileMode.CLAMP);
                mPaint.setShader(mLinearGradient);
                mGradientMatrix = new Matrix();
            }
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (mGradientMatrix != null) {
            mTranslate += mViewWidth / 5;
            if (mTranslate > 2 * mViewWidth) {
                mTranslate = -mViewWidth;
            }
            mGradientMatrix.setTranslate(mTranslate, 0);
            mLinearGradient.setLocalMatrix(mGradientMatrix);
            postInvalidateDelayed(100);
        }
    }



