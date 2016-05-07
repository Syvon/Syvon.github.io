---
layout:     post
title:      "创建自定义控件2"
subtitle:   "创建复合控件"
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


## 创建复合控件

这种方式通常需要继承一个合适的ViewGroup，再给它添加指定功能的控件，从而组合成新的复合控件。通过这种方式创建的控件，我们一般会给它指定一些可配置的属性，让它具有更强的扩展性。

下面我们创建一个TopBar。

- 定义属性

在res资源目录的values目录下创建一个attrs.xml的属性定义文件，为View提供可自定义的属性。

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <declare-styleable name="TopBar">
	        <attr name="title" format="string" />
	        <attr name="titleTextSize" format="dimension" />
	        <attr name="titleTextColor" format="color" />
	        <attr name="leftTextColor" format="color" />
	        <attr name="leftBackground" format="reference|color" />
	        <attr name="leftText" format="string" />
	        <attr name="rightTextColor" format="color" />
	        <attr name="rightBackground" format="reference|color" />
	        <attr name="rightText" format="string" />
	    </declare-styleable>
	</resources>
在代码中通过 <declare-styleable> 标签声明了使用自定义属性，并通过name属性来确定引用的名称，最后通过<attr>标签来声明具体的自定义属性。

- 创建自定义控件

创建好属性之后，就可以创建一个自定义控件-TopBar，并让它继承自ViewGroup，从而组合一些需要的控件。

	public class TopBar extends RelativeLayout {	

	    // 包含topbar上的元素：左按钮、右按钮、标题
	    private Button mLeftButton, mRightButton;
	    private TextView mTitleView;	

	    // 布局属性，用来控制组件元素在ViewGroup中的位置
	    private LayoutParams mLeftParams, mTitlepParams, mRightParams;	

	    // 左按钮的属性值，即我们在atts.xml文件中定义的属性
	    private int mLeftTextColor;
	    private Drawable mLeftBackground;
	    private String mLeftText;
	    // 右按钮的属性值，即我们在atts.xml文件中定义的属性
	    private int mRightTextColor;
	    private Drawable mRightBackground;
	    private String mRightText;
	    // 标题的属性值，即我们在atts.xml文件中定义的属性
	    private float mTitleTextSize;
	    private int mTitleTextColor;
	    private String mTitle;	

	    // 映射传入的接口对象
	    private topbarClickListener mListener;	

	    public TopBar(Context context, AttributeSet attrs, int defStyle) {
	        super(context, attrs, defStyle);
	    }	

	    public TopBar(Context context) {
	        super(context);
	    }	

	    public TopBar(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        // 设置topbar的背景
	        setBackgroundColor(0xFFF59563);
	        // 通过这个方法，将你在atts.xml中定义的declare-styleable
	        // 的所有属性的值存储到TypedArray中
	        TypedArray ta = context.obtainStyledAttributes(attrs,
	                R.styleable.TopBar);
	        // 从TypedArray中取出对应的值来为要设置的属性赋值
	        mLeftTextColor = ta.getColor(
	                R.styleable.TopBar_leftTextColor, 0);
	        mLeftBackground = ta.getDrawable(
	                R.styleable.TopBar_leftBackground);
	        mLeftText = ta.getString(R.styleable.TopBar_leftText);	

	        mRightTextColor = ta.getColor(
	                R.styleable.TopBar_rightTextColor, 0);
	        mRightBackground = ta.getDrawable(
	                R.styleable.TopBar_rightBackground);
	        mRightText = ta.getString(R.styleable.TopBar_rightText);	

	        mTitleTextSize = ta.getDimension(
	                R.styleable.TopBar_titleTextSize, 10);
	        mTitleTextColor = ta.getColor(
	                R.styleable.TopBar_titleTextColor, 0);
	        mTitle = ta.getString(R.styleable.TopBar_title);	

	        // 获取完TypedArray的值后，一般要调用
	        // recyle方法来避免重新创建的时候的错误
	        ta.recycle();	

	        mLeftButton = new Button(context);
	        mRightButton = new Button(context);
	        mTitleView = new TextView(context);	

	        // 为创建的组件元素赋值
	        // 值就来源于我们在引用的xml文件中给对应属性的赋值
	        mLeftButton.setTextColor(mLeftTextColor);
	        mLeftButton.setBackground(mLeftBackground);
	        mLeftButton.setText(mLeftText);	

	        mRightButton.setTextColor(mRightTextColor);
	        mRightButton.setBackground(mRightBackground);
	        mRightButton.setText(mRightText);	

	        mTitleView.setText(mTitle);
	        mTitleView.setTextColor(mTitleTextColor);
	        mTitleView.setTextSize(mTitleTextSize);
	        mTitleView.setGravity(Gravity.CENTER);	

	        // 为组件元素设置相应的布局元素
	        mLeftParams = new LayoutParams(
	                LayoutParams.WRAP_CONTENT,
	                LayoutParams.MATCH_PARENT);
	        mLeftParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT, TRUE);
	        // 添加到ViewGroup
	        addView(mLeftButton, mLeftParams);	

	        mRightParams = new LayoutParams(
	                LayoutParams.WRAP_CONTENT,
	                LayoutParams.MATCH_PARENT);
	        mRightParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT, TRUE);
	        addView(mRightButton, mRightParams);	

	        mTitlepParams = new LayoutParams(
	                LayoutParams.WRAP_CONTENT,
	                LayoutParams.MATCH_PARENT);
	        mTitlepParams.addRule(RelativeLayout.CENTER_IN_PARENT, TRUE);
	        addView(mTitleView, mTitlepParams);	

	        // 按钮的点击事件，不需要具体的实现，
	        // 只需调用接口的方法，回调的时候，会有具体的实现
	        mRightButton.setOnClickListener(new OnClickListener() {	

	            @Override
	            public void onClick(View v) {
	                mListener.rightClick();
	            }
	        });	

	        mLeftButton.setOnClickListener(new OnClickListener() {	

	            @Override
	            public void onClick(View v) {
	                mListener.leftClick();
	            }
	        });
	    }	

	    // 暴露一个方法给调用者来注册接口回调
	    // 通过接口来获得回调者对接口方法的实现
	    public void setOnTopbarClickListener(topbarClickListener mListener) {
	        this.mListener = mListener;
	    }	

	    /**
	     * 设置按钮的显示与否 通过id区分按钮，flag区分是否显示
	     *
	     * @param id   id
	     * @param flag 是否显示
	     */
	    public void setButtonVisable(int id, boolean flag) {
	        if (flag) {
	            if (id == 0) {
	                mLeftButton.setVisibility(View.VISIBLE);
	            } else {
	                mRightButton.setVisibility(View.VISIBLE);
	            }
	        } else {
	            if (id == 0) {
	                mLeftButton.setVisibility(View.GONE);
	            } else {
	                mRightButton.setVisibility(View.GONE);
	            }
	        }
	    }	

	    // 接口对象，实现回调机制，在回调方法中
	    // 通过映射的接口对象调用接口中的方法
	    // 而不用去考虑如何实现，具体的实现由调用者去创建
	    public interface topbarClickListener {
	        // 左按钮点击事件
	        void leftClick();
	        // 右按钮点击事件
	        void rightClick();
	    }
	}

- 实现接口回调

在调用者的代码中，调用者需要实现这样一个接口，并完成接口中的方法，确定具体的实现逻辑，并使用第二步中暴露的方法，将接口的对象传递进去，从而完成回调。通常情况下，可以使用匿名内部类的形式来实现接口中的方法。

	// 为topbar注册监听事件，传入定义的接口
	// 并以匿名类的方式实现接口内的方法
	mTopbar.setOnTopbarClickListener(
	        new TopBar.topbarClickListener() {
	            @Override
	            public void rightClick() {
	                Toast.makeText(TopBarTest.this,
	                        "right", Toast.LENGTH_SHORT)
	                        .show();
	            }
	            @Override
	            public void leftClick() {
	                Toast.makeText(TopBarTest.this,
	                        "left", Toast.LENGTH_SHORT)
	                        .show();
	            }
	        });
	// 控制topbar上组件的状态
    mTopbar.setButtonVisable(0, true);
    mTopbar.setButtonVisable(1, false);

- 引用UI模板

在需要使用的地方引用UI模板。在引用前，需要指定引用第三方控件的名字空间。

在布局文件中，可以看到这样一行代码。

	xmlns:android="http://schemas.android.com/apk/res/android"

这行代码就是在指定引用的名字空间xmlns，即xml namespace。这里指定了名字空间为“android”，因此在接下来使用系统属性的时候，才可以使用“android：”来引用Android的系统属性。

同样的，如果要使用自定义的属性，那么需要创建自己的名字空间，在Android Studio中，第三方控件都是用如下代码来引用名字空间。

	xmlns:custom="http://schemas.android.com/apk/res-auto"

这里我们将引入的第三方控件的名字空间取名为custom，之后在XML文件中使用自定义的属性时，就可以通过这个名字空间来引用。

	<com.example.systemwidget.TopBar
        android:id="@+id/topBar"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        custom:leftBackground="@drawable/blue_button"
        custom:leftText="Back"
        custom:leftTextColor="#FFFFFF"
        custom:rightBackground="@drawable/blue_button"
        custom:rightText="More"
        custom:rightTextColor="#FFFFFF"
        custom:title="自定义标题"
        custom:titleTextColor="#123412"
        custom:titleTextSize="10sp"/>

- 将UI模板写入到布局文件中

可以更进一步，将UI模板写入到布局文件中。

	<com.wun.mytopbar.Topbar xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:custom="http://schemas.android.com/apk/res-auto"
	    android:id="@+id/topBar"
	    android:layout_width="match_parent"
	    android:layout_height="40dp"
	    custom:leftBackground="@drawable/blue_button"
	    custom:leftText="Back"
	    custom:leftTextColor="#FFFFFF"
	    custom:rightBackground="@drawable/blue_button"
	    custom:rightText="More"
	    custom:rightTextColor="#FFFFFF"
	    custom:title="自定义标题"
	    custom:titleTextColor="#123412"
	    custom:titleTextSize="15sp">
	</com.wun.mytopbar.Topbar>

在其他的布局文件中，直接通过<include>标签来引用这个UI模板View。

	<include layout="@layout/topbar"/>