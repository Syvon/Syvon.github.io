---
layout:     post
title:      "创建自定义控件3"
subtitle:   "对现有控件进行扩展"
date:       2016-05-04
author:     "WunWun"
header-img: "img/in-post/android-heros/android-hello.jpg"
tags:
    - Android
    - 自定义控件
---

## 实现自定义控件

实现自定义控件的三种方法

- 对现有控件进行扩展

- 通过组合来实现新的控件

- 重写View来实现全新的控件

在View中比较重要的回调方法

- onFinishInflate() 从XML加载组件后回调

- onSizeChanged() 组件大小改变时回调

- onMeasure() 回调该方法来进行测量

- onLayout() 回调该方法确定显示的位置

- onTouchEvent() 监听到触摸事件时回调

## 重写View来实现全新的控件

创建一个自定义View，难点在于绘制控件和实现交互。通常需要继承View类，并重写它的onDraw(),onMeasure()等方法来实现绘制逻辑，同时通过重写onTouchEvent()等触控事件来实现交互逻辑。

#### 弧线展示图

![java-javascript](/img/in-post/android-heros/android-arc.png)
<small class="img-hint">比例图</small>

这个自定义View分为三部分，中间的圆形，中间显示的文字和外圈的弧线。只要在onDraw()方法中一个个去绘制就可以了。

设置圆的参数。

    mCircleXY = length / 2;
    mRadius = (float) (length * 0.5 / 2);

绘制弧线，需要指定其椭圆的的外接矩形。

    mArcRectF = new RectF(
                (float) (length * 0.1),
                (float) (length * 0.1),
                (float) (length * 0.9),
                (float) (length * 0.9));

接下来在onDraw()方法中绘制。

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // 绘制圆
        canvas.drawCircle(mCircleXY, mCircleXY, mRadius, mCirclePaint);
        // 绘制弧线
        canvas.drawArc(mArcRectF, 270, mSweepAngle, false, mArcPaint);
        // 绘制文字
        canvas.drawText(mShowText, 0, mShowText.length(),
                mCircleXY, mCircleXY + (mShowTextSize / 4), mTextPaint);
    }

提供方法让调用者设置不同的状态值。

    public void setSweepValue(float sweepValue) {
        if (sweepValue != 0) {
            mSweepValue = sweepValue;
        } else {
            mSweepValue = 25;
        }
        this.invalidate();
    }

调用者以如下的方式设置比例值。

    CircleProgressView circle = (CircleProgressView) findViewById(R.id.circle);
    circle.setSweepValue(65);

#### 音频条形图

![java-javascript](/img/in-post/android-heros/android-audio.png)
<small class="img-hint">音频条形图</small>

绘制静态音频条形图，其实就是绘制一个个的矩形，每个矩形之间稍微偏移一点距离即可。

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        for (int i = 0; i < mRectCount; i++) {
            mRandom = Math.random();
            float currentHeight = (float) (mRectHeight * mRandom);
            canvas.drawRect(
                    (float) (mWidth * 0.4 / 2 + mRectWidth * i ),
                    currentHeight,
                    (float) (mWidth * 0.4 / 2 + mRectWidth * (i + 1) - offset),
                    mRectHeight,
                    mPaint);
        }
        //invalidate();
    }

在onDraw()方法中调用invalidate()方法通知View进行重绘就可以了。不过刷新太快会影响视觉效果，因此可以使用延迟。

    postInvalidateDelayed(300);

给绘制的Paint对象增加一个LinearGradient渐变效果，这样不同高度的就会有不同颜色的渐变效果，更能够模拟音频条形图的风格。

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = getWidth();
        mRectHeight = getHeight();
        mRectWidth = (int) (mWidth * 0.6 / mRectCount);
        mLinearGradient = new LinearGradient(
                0,
                0,
                mRectWidth,
                mRectHeight,
                Color.YELLOW,
                Color.RED,
                Shader.TileMode.CLAMP);
        mPaint.setShader(mLinearGradient);
    }