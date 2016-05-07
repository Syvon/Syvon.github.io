---
layout:     post
title:      "自定义ViewGroup"
subtitle:   ""
date:       2016-05-07
author:     "WunWun"
header-img: "img/in-post/android-heros/android-viewgroup.jpg"
tags:
    - Android
    - 自定义控件
---

### 自定义ViewGroup

ViewGroup存在的目的就是为了对其子View进行管理，为其子View添加显示，响应的规则。因此自定义ViewGroup通常需要重写onMeasure()方法来对其子View进行测量，重写onLayout()方法来确定子View的位置，重写onTouchEvent()方法增加响应事件。



### 实现一个类似ScrollView的ViewGroup

1. 首先放置好ViewGroup的子View，使用遍历的方式来通知子View对自身进行测量。

	@Override
    protected void onMeasure(int widthMeasureSpec,
                             int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int count = getChildCount();
        for (int i = 0; i < count; ++i) {
            View childView = getChildAt(i);
            measureChild(childView,
                    widthMeasureSpec, heightMeasureSpec);
        }
    }

2. 确定整个ViewGroup的高度。在这里我们让每个子View占一屏的高度，整个ViewGroup的高度即子View的个数乘以屏幕的高度。

    // 设置ViewGroup的高度
    MarginLayoutParams mlp = (MarginLayoutParams) getLayoutParams();
    mlp.height = mScreenHeight * childCount;
    setLayoutParams(mlp);

3. 通过遍历的方式来设定每个子View需要放置的位置，直接通过调用子View的layout()方法，并将具体的位置作为参数传递进去即可。

	@Override
    protected void onLayout(boolean changed,
                            int l, int t, int r, int b) {
        int childCount = getChildCount();
        // 设置ViewGroup的高度
        MarginLayoutParams mlp = (MarginLayoutParams) getLayoutParams();
        mlp.height = mScreenHeight * childCount;
        setLayoutParams(mlp);
        for (int i = 0; i < childCount; i++) {
            View child = getChildAt(i);
            if (child.getVisibility() != View.GONE) {
                child.layout(l, i * mScreenHeight,
                        r, (i + 1) * mScreenHeight);
            }
        }
    }

4. 现在已经将子View放置到ViewGroup中了 ，但还不能响应任何触控事件。因此需要重写onTouchEvent()方法，为ViewGroup添加响应事件，用scrollBy()方法来辅助滑动。

	@Override
    public boolean onTouchEvent(MotionEvent event) {
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mLastY = y;
                mStart = getScrollY();
                break;
            case MotionEvent.ACTION_MOVE:
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                int dy = mLastY - y;
                if (getScrollY() < 0) {
                    dy = 0;
                }
                if (getScrollY() > getHeight() - mScreenHeight) {
                    dy = 0;
                }
                scrollBy(0, dy);
                mLastY = y;
                break;
            
        }
        postInvalidate();
        return true;
    }

5. 在ACTION_UP事件中判断手指滑动的距离，如果超过一定距离，则使用Scroller类来平滑到下一个子View；如果小于一定距离，则回滚到原来的位置。

	case MotionEvent.ACTION_UP:
    int dScrollY = checkAlignment();
    if (dScrollY > 0) {
        if (dScrollY < mScreenHeight / 3) {
            mScroller.startScroll(
                    0, getScrollY(),
                    0, -dScrollY);
        } else {
            mScroller.startScroll(
                    0, getScrollY(),
                    0, mScreenHeight - dScrollY);
        }
    } else {
        if (-dScrollY < mScreenHeight / 3) {
            mScroller.startScroll(
                    0, getScrollY(),
                    0, -dScrollY);
        } else {
            mScroller.startScroll(
                    0, getScrollY(),
                    0, -mScreenHeight - dScrollY);
        }
    }
    break;

