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

## 屏幕的尺寸信息

#### 屏幕参数

- 屏幕大小

指屏幕对角线的长度，通常使用“寸”来度量，例如4.7寸，5.5寸

- 分辨率

分辨率指手机屏幕的像素点个数，例如720*1280就是指屏幕的分辨率。指宽有720个像素点，而高有1280个像素点。

- PPI

每英寸像素（Pixels Per Inch），又被称为DPI（Dots Per Inch）。它是由对角线的像素点数除以屏幕的大小得到的。

#### 独立像素密度dp

各种屏幕密度的不同，导致同样像素大小的长度，在不同密度的屏幕上显示长度不同。

Android系统使用mdpi即密度值为160的屏幕作为标准，在这个屏幕上 1px = 1dp

各个分辨率直接的换算关系 ldpi:mdpi:hdpi:shdpi:xxhdpi=3:4:6:8:12

例如，在mdpi中1dp=1px，在hdpi中1dp=1.5px，在xhdpi中1dp=2px，在xxhdpi中1dp=3px

#### 单位转换

	public class DisplayUtil {	

	    private static float scale;	

	    /**
	     * 根据手机的分辨率从 dp 的单位 转成为 px(像素)
	     */
	    public static int dip2px(Context context, float dpValue) {
	        if (scale == 0) {
	            scale = context.getResources().getDisplayMetrics().density;
	        }
	        return (int) (dpValue * scale + 0.5f);
	    }	

	    /**
	     * 根据手机的分辨率从 px(像素) 的单位 转成为 dp
	     */
	    public static int px2dip(Context context, float pxValue) {
	        if (scale == 0) {
	            scale = context.getResources().getDisplayMetrics().density;
	        }
	        return (int) (pxValue / scale + 0.5f);
	    }
	}

## Android XML 绘图

#### Bitmap

	<?xml version="1.0" encoding="utf-8"?>
	<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@drawable/ic_launcher">
	</bitmap>

#### Shape

通过Shape可以在XML中绘制各种形状

	<?xml version="1.0" encoding="utf-8"?>
	<shape xmlns:android="http://schemas.android.com/apk/res/android" 
		// 此处可以设置shape的形状 不设置默认为rectangle
		android:shape=["rectangle"|"oval"|"line"|"ring"|]>	

	    <!-- 圆角 shape="rectangle"是使用 默认为1dp -->
	    <corners
	        android:radius="xdp"
	        android:topLeftRadius="xdp"
	        android:topRightRadius="xdp"
	        android:bottomLeftRadius="xdp"
	        android:bottomRightRadius="xdp"/>	

	    <!-- 渐变 -->
	    <gradient
	        android:startColor="color"
	        android:centerColor="color"
	        android:endColor="color"
	        android:useLevel="boolean"
	        android:angle="integer"//angle的值必须是45的倍数（包括0），仅在type="linear"有效
	        android:type=["linear"|"radial"|"sweep"]
	        android:centerX="integer"
	        android:centerY="integer"
	        android:gradientRadius="integer"/>	

	    <!-- 间隔 -->
	    <padding
	        android:left="xdp"
	        android:top="xdp"
	        android:right="xdp"
	        android:bottom="xdp"/>	

	    <!-- 大小 宽度和高度 -->
	    <size
	        android:width="dp"
	        android:height="dp"/>	

	    <!-- 填充 -->
	    <solid
	        android:color="color"/><!-- 填充的颜色 -->	

	    <!-- 描边 -->
	    <stroke
	        android:width="dp"
	        android:color="color"
	        android:dashWidth="dp" //虚线宽度
	        android:dashGap="dp"/> //虚线间隔宽度	

	</shape>

#### layer

layer有点类似PS中的图层

	<?xml version="1.0" encoding="utf-8"?>  
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android">  
	    <!-- item1 -->
	    <item>  
	      <bitmap android:src="drawable"  
	        android:gravity="center" />  
	    </item>  
	    <!-- item2 -->
	    <item>  
	      <bitmap android:src="drawable"  
	        android:gravity="center" />  
	    </item>  
	    <!-- item3 -->
	    <item 
	      <bitmap android:src="drawable"  
	        android:gravity="center" />  
	    </item>  
	</layer-list>

#### Seclector

Seclector的作用在于帮助开发者实现静态绘图中的事件反馈，通过给不同的事件设置不同的图像，从而在程序中根据用户输入，返回不同的效果。

	<?xml version="1.0" encoding="utf-8" ?>     
		<selector xmlns:Android="http://schemas.android.com/apk/res/android">   
		<!-- 默认时的背景图片-->    
		<item Android:drawable="drawable" />      
		<!-- 没有焦点时的背景图片 -->    
		<item 
		   Android:state_window_focused="false"      
		   android:drawable="drawable" 
		   />     
		<!-- 非触摸模式下获得焦点并单击时的背景图片 -->    
		<item 
		   Android:state_focused="true" 
		   android:state_pressed="true"   
		   android:drawable= "drawable" 
		   />   
		<!-- 触摸模式下单击时的背景图片-->    
		<item 
		   Android:state_focused="false" 
		   Android:state_pressed="true"   
		   Android:drawable="drawable" 
		   />    
		<!--选中时的图片背景-->    
		<item 
		   Android:state_selected="true" 
		   android:drawable="drawable" 
		   />     
		<!--获得焦点时的图片背景-->    
		<item 
		   Android:state_focused="true" 
		   Android:drawable="drawable" 
		   />     
	</selector>  

## Android绘图机巧

#### Canvas

- Canvas.save()      将之前的所有已绘制图像保存起来，让后续的操作就好像在一个新的图层上操作一样。

- Canvas.restore()   将save()之后绘制的所有图像与save()之前的图像进行合并

- Canvas.translate() 坐标平移

- Canvas.rotate()    坐标旋转

#### Layer图层

Android中通过调用saveLayer()方法，saveLayerAlpha()方法将一个图层入栈，使用restore()方法，restoreToCount()方法将一个图层出栈。入栈的时候，后面的所有操作都发生在这个图层上，而出栈的时候，则会把图像绘制到上层Canvas上。