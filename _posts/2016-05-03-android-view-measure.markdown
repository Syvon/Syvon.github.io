---
layout:     post
title:      "View的测量"
subtitle:   ""
date:       2016-05-03
author:     "WunWun"
header-img: "img/in-post/android-heros/android-view-measure.png"
tags:
    - Android
    - View
---

### MeasureSpec类与测量模式

Android中提供了一个MeasureSpec类，来帮助我们测量View。MeasureSpec是一个32位的int值，其中高2位为测量的模式，低30位为测量的大小。（使用位运算是为了提高并优化效率）。

测量模式分为三种。

- EXACTLY

精确模式。当把属性（layout_width或layout_height）指定为具体数值，如**android:layout_width="100dp"**,或指定为**match_parent**属性时，系统使用的EXACTLY模式。

- AT_MOST

最大值模式。当把属性指定为**wrap_content**时，控件大小一般随着子控件或内容的变化而变化。

- UNSPECIFIED

不指定其大小测量模式，View想多大就多大，通常情况下在绘制自定义View时才会使用。

View类的onMeasure()方法只支持EXACTLY模式，所以如果在自定义控件的时候不重写onMeasure()方法，就只能使用EXACTLY模式。如果要让自定义View支持wrap_content属性，那么就必须重写onMeasure()方法来指定wrp_content时的大小。

###自己动手实现一个自定义View

####View是如何进行测量的

查看View的源代码，可以知道系统最终会调用setMeasuredDimension()方法将测量后的宽高值设置进去，完成测量工作。

	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
	                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
	    }

参照View的方式，我们也在自定义的View中，调用setMeasuredDimension()方法设置宽高值。

	@Override
    protected void onMeasure(int widthMeasureSpec,
                             int heightMeasureSpec) {
        setMeasuredDimension(
                measureWidth(widthMeasureSpec),
                measureHeight(heightMeasureSpec));
    }

其中，我们调用了measureWidth( )和measureHeight( )方法，分别对宽，高进行重定义。

	private int measureWidth(int measureSpec) {
	        int result = 0;
	        int specMode = MeasureSpec.getMode(measureSpec);
	        int specSize = MeasureSpec.getSize(measureSpec);	

	        if (specMode == MeasureSpec.EXACTLY) {
	            result = specSize;
	        } else {
	            result = 200;
	            if (specMode == MeasureSpec.AT_MOST) {
	                result = Math.min(result, specSize);
	            }
	        }
	        return result;
	    }

当指定宽高属性为wrap_content属性时，如果不重写onMeasure()方法，那么系统就不知道该使用默认多大的尺寸，会默认填充整个父布局。所以重写onMeasure()方法的目的，就是为了能够给View一个wrap_content属性下的默认大小。

