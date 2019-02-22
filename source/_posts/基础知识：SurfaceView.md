---
title: 基础知识：SurfaceView
copyright: true
date: 2019-02-22 17:32:25
tags:
- SurfaceView
categories:
- Java
---

#### 1. SurfaceView与View的区别

- View主要适用于主动更新的情况，而SurfaceView主要适用于被动更新，例如频繁的刷新。
- View在主线程中对页面进行刷新，而SurfaceView通常会通过一个子线程来进行页面的刷新。
- View在绘图时没有使用双缓冲机制，而SurfaceView在底层的实现机制中就已经实现了双缓冲机制。

<!-- more -->

#### 2.什么是双缓冲机制？

双缓冲可以理解为有两个线程轮番去解析视频流的帧图像，当一个线程解析完帧图像后，把图像渲染到界面中，同时另一线程开始解析下一帧图像，使得两个线程轮番配合去解析视频流，以达到流畅播放的效果。

#### 3.SurfaceView模板使用

通常我们使用SurfaceView时可以使用如下的模板进行扩充，重点是继承SurfaceView，实现SurfaceHolder.Callback和Runnable接口。

下面的LineSurfaceView实现了两种效果 :

```java
package com.exalple.wudu.view;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Path;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

public class LineSurfaceView extends SurfaceView implements SurfaceHolder.Callback,Runnable{
    private SurfaceHolder holder;
    private Canvas canvas;
    private boolean isDrawing;

    private Path path;
    private Paint paint;
    private int x ,y;

    public LineSurfaceView(Context context) {
        super(context);
        initView();
    }

    public LineSurfaceView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public LineSurfaceView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        holder = getHolder();
        holder.addCallback(this);

        setFocusable(true);
        setFocusableInTouchMode(true);
        this.setKeepScreenOn(true);

        path = new Path();
        paint = new Paint(Paint.ANTI_ALIAS_FLAG);
        paint.setStrokeWidth(10);
        paint.setColor(Color.RED);
        //这个参数会控制画出的图形是不是线条还是扇形
        paint.setStyle(Paint.Style.STROKE);
        paint.setStrokeCap(Paint.Cap.ROUND);
        paint.setStrokeJoin(Paint.Join.ROUND);
    }


    @Override
    public void surfaceCreated(SurfaceHolder surfaceHolder) {
        isDrawing = true;
        path.moveTo(0,400);
        new Thread(this).start();

    }

    @Override
    public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {


    }

    @Override
    public void surfaceDestroyed(SurfaceHolder surfaceHolder) {
        isDrawing = false;

    }

    @Override
    public void run() {
        long start = System.currentTimeMillis();

        while (isDrawing){
            //绘画板
            draw();
            //------放开这里的注释会自动绘制正弦函数
            //x +=1;
            //y = (int) (100 * Math.sin(x * 2 * Math.PI / 180) + 400);
            //path.lineTo(x,y);
        }

        long end = System.currentTimeMillis();

        //50-100:经验值，一般是最合适的
        if(end - start < 100){
            try {
                Thread.sleep(100 - (end - start));
            }catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int dx = (int) event.getX();
        int dy  = (int) event.getY();


        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                path.moveTo(dx,dy);
                break;
            case MotionEvent.ACTION_MOVE:
                path.lineTo(dx,dy);
                break;
            case MotionEvent.ACTION_UP:

                break;
        }
        return true;
    }

    private void draw() {
        try {
            canvas = holder.lockCanvas();
            canvas.drawColor(Color.WHITE);
            canvas.drawPath(path,paint);

        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if(canvas != null){
                holder.unlockCanvasAndPost(canvas);
            }
        }


    }
}
```

- 效果1；未放开run方法中的注释，实现的是画板效果，可以手动绘制内容。

  ![5c6fd0e6ee2d2](https://i.loli.net/2019/02/22/5c6fd0e6ee2d2.gif)

- 效果2：放开run方法中的注释，实现的是自动绘制正弦函数的效果。

  ![5c6fd0fb2efbc](https://i.loli.net/2019/02/22/5c6fd0fb2efbc.gif)

#### 4.事件分发

- onTouchEvent

  > 用来处理事件
  > 
  > 返回true，表示该View可以处理好此事件，再向上传递，即不需要父类再来处理这个事件了。
  > 
  > 返回false，表示该View不能处理该事件，需要传递给父类的onTouchEvent方法来处理。

- dispatchTouchEvent

  > 用来分派事件
  > 
  > 其中调用了onInterceptTouchEvent和onTouchEvent方法，一般不重写该方法

- onInterceptTouchEvent

  > 用来拦截事件
  > 
  > ViewGroup类中的源码实现就是{return false;}表示不拦截该事件，事件将向下传递()传递给其子View)；
  > 
  > 若手动重写该方法，使其返回true则表示拦截，事件将终止向下传递，事件由当前ViewGroup类来处理，就是调用该类的onTouchEvent()方法。
