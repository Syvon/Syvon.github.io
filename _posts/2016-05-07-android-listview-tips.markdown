---
layout:     post
title:      "ListView使用技巧"
subtitle:   ""
date:       2016-05-07
author:     "WunWun"
header-img: "img/android-rain.jpg"
tags:
    Android
    ListView
---

## ListView常用优化技巧

#### 使用ViewHolder模式提高效率   

ViewHolder模式充分利用ListView的视图缓存机制，避免了每次在调用getView()的时候都去通过findViewById()实例化控件。使用ViewHolder非常简单，只需要在自定义Adapter中定义一个内部类ViewHolder，并将布局中的控件作为成员变量。   

#### 设置项目间分隔线 

ListView各个项目之间可以通过设置分割线来区分，系统提供了divider和dividerHeight这样两个属性来帮助我们实现这一功能。 

    android:divider="android:color/darker_gray"
    android:dividerHeight="10dp"    

特殊情况下，当设置分隔线为null时，分隔线就为透明的。    

    android:divider="null"  

#### 隐藏ListView的滚动条   

设置scrollbars属性为none的时候，就不会出现滚动条。    

    android:scrollbars="none"   

#### 取消ListView的Item点击效果  

可以通过修改listSelector属性来取消掉点击后的效果  

    android:listSelector="#00000000"    

#### 设置ListView显示在第几项 

当需要指定具体显示的Item时，可以通过如下代码实现。 

    listView.setSelection(N);   

该方法类似scrollTo，是瞬间完成的。除此之外，还可以通过如下代码实现平滑移动。  

    listView.smoothScrollBy(distance,duration);
    listView.smoothScrollByOffset(offset);
    listView.smoothScrollToPosition(index); 

#### 动态修改ListView 

    mData.add("new data");
    mAdapter.notifyDataSetChanged();    

#### 遍历ListView中的所有item   

ListView作为一个ViewGroup，为我们提供了操纵子View的各种方法。   

    for(int i=0;i<mListView.getChildCount(),i++) {
        View view = mListView.getChildAt(i);
    }   

#### ListView滑动监听 

##### OnTouchListener 

OnTouchListner是View中的监听事件，通过监听ACTION_DOWN，ACTION_MOVE，ACTION_UP这三个事件发生的时的坐标，就可以根据坐标判断用户滑动的方向，并在不同的条件下进行相应的逻辑处理。 

    mListView.setOnTouchListener(new OnTouchListener() {
          @Override
          public boolean onTouch(View v, MotionEvent event) {
              switch (event.getAction()) {
              case MotionEvent.ACTION_DOWN:
                  // 触摸时操作
                  break;
              case MotionEvent.ACTION_MOVE:
                  // 移动时操作
                  break;
              case MotionEvent.ACTION_UP:
                  // 离开时操作
                  break;
              }
              return false;
          }
      });   

##### OnScrollListener    

OnScrollListener是ABSListView中的监听事件。 

    listView.setOnScrollListener(new OnScrollListener() {
          @Override
          public void onScrollStateChanged(AbsListView view, int scrollState) {
              switch (scrollState) {
              case OnScrollListener.SCROLL_STATE_IDLE:
                  //滑动停止时
                  break;
              case OnScrollListener.SCROLL_STATE_TOUCH_SCROLL:
                  //正在滚动时
                  break;
              case OnScrollListener.SCROLL_STATE_FLING:
                  //手指快速滑动，手指离开后会因惯性继续滑动时
                  break;
              }
          }
          @Override
          public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
              //firstVisibleItem 当前能看到的第一个item的id，从0开始，包括显示不完整的
              //visibleItemCount 当前能看见的item总数，包括显示不完整的
              //totalItemCount listview的item总数
              //滚动时一直调用
              if(firstVisibleItem+visibleItemCount==totalItemCount&&totalItemCount>0){
                  //滚动至最后一行
              }
              if(firstVisibleItem> lastVisibleItem){
                  //上滑
              }else if(firstVisibleItem< lastVisibleItem){
                  //下滑
              }
              lastVisibleItem=firstVisibleItem;//自行定义一个lastVisibleItem的成员变量
                              listView.getFirstVisiblePosition();
              listView.getLastVisiblePosition();//当然也可以通过ListView提供的方法直接获取
          }
      });

## ListView常用扩展

#### 自动隐藏 Toolbar

首先需要给ListView增加一个HeaderView，避免第一个Item被Toolbar遮挡。通过R.dimen.abc_action_bar_default_height_material属性获取Actionbar的高度，并设置给HeaderView。

    View header = new View(this);
    header.setLayoutParams(new AbsListView.LayoutParams(
            AbsListView.LayoutParams.MATCH_PARENT,
            (int) getResources().getDimension(
                    R.dimen.abc_action_bar_default_height_material)));
    mListView.addHeaderView(header);

监听ListView的滑动，通过比较与上次坐标的大小，来判断滑动的方向，并通过滑动的方向来判断是否需要显示或隐藏对应的布局。

    View.OnTouchListener myTouchListener = new View.OnTouchListener() {
        @Override
        public boolean onTouch(View v, MotionEvent event) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    mFirstY = event.getY();
                    break;
                case MotionEvent.ACTION_MOVE:
                    mCurrentY = event.getY();
                    if (mCurrentY mFirstY > mTouchSlop) {
                        direction = 0;// down
                    } else if (mFirstY mCurrentY > mTouchSlop) {
                        direction = 1;// up
                    }
                    if (direction == 1) {
                        if (mShow) {
                            toolbarAnim(1);//show
                            mShow = !mShow;
                        }
                    } else if (direction == 0) {
                        if (!mShow) {
                            toolbarAnim(0);//hide
                            mShow = !mShow;
                        }
                    }
                    break;
                case MotionEvent.ACTION_UP:
                    break;
            }
            return false;
        }
    };

控制布局显示隐藏的动画。

    private void toolbarAnim(int flag) {
        if (mAnimator != null && mAnimator.isRunning()) {
            mAnimator.cancel();
        }
        if (flag == 0) {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(), 0);
        } else {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(),
                    -mToolbar.getHeight());
        }
        mAnimator.start();
    }
