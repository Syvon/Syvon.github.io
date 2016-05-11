---
layout:     post
title:      "Android Scroll分析2"
subtitle:   "实现滑动的七种方法"
date:       2016-05-08
author:     "WunWun"
header-img: "img/in-post/android-heros/android-scroll.jpg"
tags:
    - Android
    - 滑动
---

滑动的基本思想：当触摸View时，系统记下当前触摸点坐标；当手指移动时，系统记下移动后的触摸点坐标，从而获取到相对于前一次坐标点的偏移量，并通过偏移量来修改View的坐标，这样不断重复，从而实现滑动过程。

### layout方法

在View绘制时，会调用onLayout()方法来设置显示的位置。同样，可以通过修改View的left，top，right，bottom四个属性来控制View的坐标。

    // 视图坐标方式
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = x;
                lastY = y;
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                // 在当前left、top、right、bottom的基础上加上偏移量
                layout(getLeft() + offsetX,
                        getTop() + offsetY,
                        getRight() + offsetX,
                        getBottom() + offsetY);

                break;
        }
        return true;
    }

    // 绝对坐标方式
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int rawX = (int) (event.getRawX());
        int rawY = (int) (event.getRawY());
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = rawX;
                lastY = rawY;
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = rawX - lastX;
                int offsetY = rawY - lastY;
                // 在当前left、top、right、bottom的基础上加上偏移量
                layout(getLeft() + offsetX,
                        getTop() + offsetY,
                        getRight() + offsetX,
                        getBottom() + offsetY);
                // 重新设置初始坐标
                lastX = rawX;
                lastY = rawY;
                break;
        }
        return true;
    }

### offsetLeftAndRight()与offsetTopAndBottom()

这个方法相当于提供的一个对左右，上下移动的API的封装。当计算出偏移量后，只需要使用如下代码就可以完成View的重新布局。

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = x;
                lastY = y;
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = x - lastX;
                int offsetY = y - lastY;

                offsetLeftAndRight(offsetX);
                offsetTopAndBottom(offsetY);
                break;
        }
        return true;
    }

### LayoutParams

LayoutParams保存了一个View的布局参数。可以在程序中，通过改变LayoutParams来动态的修改布局的位置参数，从而达到改变View位置的效果。

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = (int) event.getX();
                lastY = (int) event.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
                //LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
                layoutParams.leftMargin = getLeft() + offsetX;
                layoutParams.topMargin = getTop() + offsetY;
                setLayoutParams(layoutParams);
                break;
        }
        return true;
    }

### scrollTo()与scrollBy()

scrollTo()与scrollBy()移动的时View的content，即让View的内容移动。如果在ViewGroup中使用，那么移动的是所有子View；如果在View中使用，那么移动的是View的内容（例如TextView，content就是它的内容）

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                lastX = (int) event.getX();
                lastY = (int) event.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                ((View) getParent()).scrollBy(-offsetX, -offsetY);
                break;
        }
        return true;
    }

### Scroller

- 初始化Scroller

初始化Scroller

    private Scroller mScroller;
    //初始化Scroller
    mScroller = new Scroller(context);

- 重写computeScroll()方法，实现模拟滑动

系统在绘制View的时候，会在draw方法中调用该方法。

    @Override
    public void computeScroll() {
        super.computeScroll();
        // 判断Scroller是否执行完毕
        if (mScroller.computeScrollOffset()) {
            ((View) getParent()).scrollTo(
                    mScroller.getCurrX(),
                    mScroller.getCurrY());
            // 通过重绘来不断调用computeScroll
            invalidate();
        }
    }

- startScroll开启模拟过程

手指离开时，执行滑动过程

    case MotionEvent.ACTION_UP:
        // 手指离开时，执行滑动过程
        View viewGroup = ((View) getParent());
        mScroller.startScroll(
                viewGroup.getScrollX(),
                viewGroup.getScrollY(),
                -viewGroup.getScrollX(),
                -viewGroup.getScrollY());
        invalidate();
        break;

### 属性动画

### ViewDragHelper

    public class DragViewGroup extends FrameLayout {    

        private ViewDragHelper mViewDragHelper;
        private View mMenuView, mMainView;
        private int mWidth; 

        public DragViewGroup(Context context) {
            super(context);
            initView();
        }   

        public DragViewGroup(Context context, AttributeSet attrs) {
            super(context, attrs);
            initView();
        }   

        public DragViewGroup(Context context,
                             AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
            initView();
        }   

        @Override
        protected void onFinishInflate() {
            super.onFinishInflate();
            mMenuView = getChildAt(0);
            mMainView = getChildAt(1);
        }   

        @Override
        protected void onSizeChanged(int w, int h, int oldw, int oldh) {
            super.onSizeChanged(w, h, oldw, oldh);
            mWidth = mMenuView.getMeasuredWidth();
        }   

        @Override
        public boolean onInterceptTouchEvent(MotionEvent ev) {
            return mViewDragHelper.shouldInterceptTouchEvent(ev);
        }   

        @Override
        public boolean onTouchEvent(MotionEvent event) {
            //将触摸事件传递给ViewDragHelper,此操作必不可少
            mViewDragHelper.processTouchEvent(event);
            return true;
        }   

        private void initView() {
            mViewDragHelper = ViewDragHelper.create(this, callback);
        }   

        private ViewDragHelper.Callback callback =
                new ViewDragHelper.Callback() { 

                    // 何时开始检测触摸事件
                    @Override
                    public boolean tryCaptureView(View child, int pointerId) {
                        //如果当前触摸的child是mMainView时开始检测
                        return mMainView == child;
                    }   

                    // 触摸到View后回调
                    @Override
                    public void onViewCaptured(View capturedChild,
                                               int activePointerId) {
                        super.onViewCaptured(capturedChild, activePointerId);
                    }   

                    // 当拖拽状态改变，比如idle，dragging
                    @Override
                    public void onViewDragStateChanged(int state) {
                        super.onViewDragStateChanged(state);
                    }   

                    // 当位置改变的时候调用,常用与滑动时更改scale等
                    @Override
                    public void onViewPositionChanged(View changedView,
                                                      int left, int top, int dx, int dy) {
                        super.onViewPositionChanged(changedView, left, top, dx, dy);
                    }   

                    // 处理垂直滑动
                    @Override
                    public int clampViewPositionVertical(View child, int top, int dy) {
                        return top;
                    }   

                    // 处理水平滑动
                    @Override
                    public int clampViewPositionHorizontal(View child, int left, int dx) {
                        return left;
                    }   

                    // 拖动结束后调用
                    @Override
                    public void onViewReleased(View releasedChild, float xvel, float yvel) {
                        super.onViewReleased(releasedChild, xvel, yvel);
                        //手指抬起后缓慢移动到指定位置
                        if (mMainView.getLeft() < 500) {
                            //关闭菜单
                            //相当于Scroller的startScroll方法
                            mViewDragHelper.smoothSlideViewTo(mMainView, 0, 0);
                            ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
                        } else {
                            //打开菜单
                            mViewDragHelper.smoothSlideViewTo(mMainView, 300, 0);
                            ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
                        }
                    }
                };  

        @Override
        public void computeScroll() {
            if (mViewDragHelper.continueSettling(true)) {
                ViewCompat.postInvalidateOnAnimation(this);
            }
        }
    }