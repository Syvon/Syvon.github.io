---
layout:     post
title:      "SurfaceView"
subtitle:   ""
date:       2016-05-12
author:     "WunWun"
header-img: "img/in-post/android-heros/android-surfaceview.jpg"
tags:
    - Android
    - SurfaceView
---

## SurfaceView与View的区别

- View主要适用于主动更新的情况下；SurfaceView主要适用于被动更新，例如频繁地刷新

- View在主线程中对画面进行刷新；SurfaceView通过一个子线程进行页面刷新

- View在绘图时没有使用双缓冲机制；SurfaceView在底层实现机制中就已经实现了双缓冲机制

总结：如果自定义View需要频繁刷新，或刷新时数据处理量比较大，就可以考虑使用SurfaceView

## SurfaceView的使用模板

	public class SurfaceViewTemplate extends SurfaceView
	        implements SurfaceHolder.Callback, Runnable {	

	    // SurfaceHolder
	    private SurfaceHolder mHolder;
	    // 用于绘图的Canvas
	    private Canvas mCanvas;
	    // 子线程标志位
	    private boolean mIsDrawing;	

	    public SurfaceViewTemplate(Context context) {
	        super(context);
	        initView();
	    }	

	    public SurfaceViewTemplate(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        initView();
	    }	

	    public SurfaceViewTemplate(Context context, AttributeSet attrs, int defStyle) {
	        super(context, attrs, defStyle);
	        initView();
	    }	

	    private void initView() {
	        mHolder = getHolder();
	        mHolder.addCallback(this);
	        setFocusable(true);
	        setFocusableInTouchMode(true);
	        this.setKeepScreenOn(true);
	        //mHolder.setFormat(PixelFormat.OPAQUE);
	    }	

	    @Override
	    public void surfaceCreated(SurfaceHolder holder) {
	        mIsDrawing = true;
	        new Thread(this).start();
	    }	

	    @Override
	    public void surfaceChanged(SurfaceHolder holder,
	                               int format, int width, int height) {
	    }	

	    @Override
	    public void surfaceDestroyed(SurfaceHolder holder) {
	        mIsDrawing = false;
	    }	

	    @Override
	    public void run() {
	        while (mIsDrawing) {
	            draw();
	        }
	    }	

	    private void draw() {
	        try {
	            mCanvas = mHolder.lockCanvas();
	            // draw sth
	        } catch (Exception e) {
	        } finally {
	            if (mCanvas != null)
	                mHolder.unlockCanvasAndPost(mCanvas);
	        }
	    }
	}

## 利用SurfaceView实现正弦曲线的实例

	public class SinView extends SurfaceView
	        implements SurfaceHolder.Callback, Runnable {	

	    private SurfaceHolder mHolder;
	    private Canvas mCanvas;
	    private boolean mIsDrawing;
	    private int x = 0;
	    private int y = 0;
	    private Path mPath;
	    private Paint mPaint;	

	    public SinView(Context context) {
	        super(context);
	        initView();
	    }	

	    public SinView(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        initView();
	    }	

	    public SinView(Context context, AttributeSet attrs, int defStyle) {
	        super(context, attrs, defStyle);
	        initView();
	    }	

	    private void initView() {
	        mHolder = getHolder();
	        mHolder.addCallback(this);
	        setFocusable(true);
	        setFocusableInTouchMode(true);
	        this.setKeepScreenOn(true);
	        mPath = new Path();
	        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
	        mPaint.setColor(Color.RED);
	        mPaint.setStyle(Paint.Style.STROKE);
	        mPaint.setStrokeWidth(10);
	        mPaint.setStrokeCap(Paint.Cap.ROUND);
	        mPaint.setStrokeJoin(Paint.Join.ROUND);
	    }	

	    @Override
	    public void surfaceCreated(SurfaceHolder holder) {
	        mIsDrawing = true;
	        mPath.moveTo(0, 400);
	        new Thread(this).start();
	    }	

	    @Override
	    public void surfaceChanged(SurfaceHolder holder,
	                               int format, int width, int height) {
	    }	

	    @Override
	    public void surfaceDestroyed(SurfaceHolder holder) {
	        mIsDrawing = false;
	    }	

	    @Override
	    public void run() {
	        while (mIsDrawing) {
	            draw();
	            x += 1;
	            y = (int) (100*Math.sin(x * 2 * Math.PI / 180) + 400);
	            mPath.lineTo(x, y);
	        }
	    }	

	    private void draw() {
	        try {
	            mCanvas = mHolder.lockCanvas();
	            // SurfaceView背景
	            mCanvas.drawColor(Color.WHITE);
	            mCanvas.drawPath(mPath, mPaint);
	        } catch (Exception e) {
	        } finally {
	            if (mCanvas != null)
	                mHolder.unlockCanvasAndPost(mCanvas);
	        }
	    }
	}