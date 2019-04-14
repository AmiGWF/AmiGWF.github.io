---
title: 自定义View方法查询手册-Canvas
copyright: true
date: 2019-04-14 11:00:46
tags:
- Canvas
categories:
- Java
---

**系列文章**

[1. 自定义View方法查询手册-Paint](https://www.syncxiao.com/2019/03/07/%E8%87%AA%E5%AE%9A%E4%B9%89View%E6%96%B9%E6%B3%95%E6%9F%A5%E8%AF%A2%E6%89%8B%E5%86%8C-Paint)

[2. 自定义View方法查询手册-Canvas](https://www.syncxiao.com/2019/04/14/%E8%87%AA%E5%AE%9A%E4%B9%89View%E6%96%B9%E6%B3%95%E6%9F%A5%E8%AF%A2%E6%89%8B%E5%86%8C-Canvas)


---

<!-- more -->


**1. 实心圆**
![](https://i.loli.net/2019/04/14/5cb2bc8fd362d.jpg)

```java
paint.setStyle(Paint.Style.FILL);
//参数分别表示：圆心X，圆心Y，圆形半径，画笔
canvas.drawCircle(300,300,200,paint);
```

**2. 空心圆**
![Alt text](https://i.loli.net/2019/04/14/5cb2bc8fe509d.jpg)

```java
paint2.setStrokeWidth(30);
paint.setStyle(Paint.Style.STROKE);
//参数分别表示：圆心X，圆心Y，圆形半径，画笔
canvas.drawCircle(300,300,200,paint);
```

**3. 直角矩形**
![Alt text](https://i.loli.net/2019/04/14/5cb2bc8fe6137.jpg)

```java
//参数分别表示：left、top、right、bottom，即左上角、右下角坐标的X、Y，画笔
canvas.drawRect(100,100,600,400,paint);
```

**4. 直角矩形 - - Rect**
![Alt text](https://i.loli.net/2019/04/14/5cb2bc90045d6.jpg)

```java
//创建Rect对象，分别设置左上角、右下角坐标
Rect rect = new Rect();
rect.left = 100;
rect.top = 100;
rect.right = 600;
rect.bottom = 400;
canvas.drawRect(rect,paint);
```

**5. 圆角矩形**
![Alt text](https://i.loli.net/2019/04/14/5cb2beca6dd8a.jpg)

```java
RectF rect = new RectF();
rect.left = 100;
rect.top = 100;
rect.right = 600;
rect.bottom = 400;
//参数分别表示：矩形、X轴、Y轴方向的圆角半径、画笔
canvas.drawRoundRect(rect,20,20,paint);
```

**6. 小圆点、小方块**
![Alt text](https://i.loli.net/2019/04/14/5cb2bc9005bbb.jpg)

```java
//设置小圆点的大小
paint.setStrokeWidth(50);
//平头小方块
paint.setStrokeCap(Paint.Cap.BUTT);
//参数分别是X、Y点坐标
canvas.drawPoint(200,400,paint);
//小圆点
paint.setStrokeCap(Paint.Cap.ROUND);
canvas.drawPoint(300,400,paint);
//方头小方块
paint.setStrokeCap(Paint.Cap.SQUARE);
canvas.drawPoint(400,400,paint);
```

**7. 连续小圆点、小方块**
![Alt text](https://i.loli.net/2019/04/14/5cb2bc9005d85.jpg)

> **pts：**坐标数组
> **offset：**跳过数组中前面几个数，才开始计算圆点坐标
> **count：**一起要多少个数组中的数来进行坐标圆点绘制，两个数算作一个点，越界会出现异常，单个数不算作一个点
> **drawPoints(float[] pts, int offset, int count, Paint paint)**

```java
drawPoints(float[] pts, int offset, int count, @RecentlyNonNull Paint paint)

float[] points = new float[]{100,100,200,100,300,100,400,100};
canvas.drawPoints(points,paint);

float[] points2 = new float[]{100,300,200,300,300,300,400,300};
canvas.drawPoints(points2,1,6,paint);
```
**8. 椭圆**
![Alt text](https://i.loli.net/2019/04/14/5cb2bc9006b96.jpg)

```java
//left、top、right、bottom位置的坐标
canvas.drawOval(100,100,600,400,paint);

RectF rect = new RectF();
rect.left = 100;
rect.top = 100;
rect.right = 600;
rect.bottom = 400;
canvas.drawOval(rect,paint);
```

**9. 画线条**
![Alt text](https://i.loli.net/2019/04/14/5cb2bc90069a3.jpg)

> **pts：**坐标数组
> **offset：**跳过数组中前面几个数，才开始计算直线的坐标
> **count：**一起要多少个数组中的数来进行直线坐标的绘制，两个数算作一个点，直线需要两个点，也就是四个数构成一条直线，越界会出现异常，少于四个数不算作一条直线
> **drawLines(float[] pts, int offset, int count, Paint paint)**

```java
 canvas.drawLine(100,100,400,100,paint);

//画一个线条组成的正方形
float[] lines = new float[]{100,200,500,200,500,200,500,500,500,500,100,500,100,500,100,200};
canvas.drawLines(lines,paint);
```

**10. 扇形**
![Alt text](https://i.loli.net/2019/04/14/5cb2cbd3dadc7.jpg)

> **oval：**矩形
> **startAngle：**扇形从哪个角度开始(顺时针0-360)
> **sweepAngle：**扇形整个展开的角度
> **useCenter：**是否连接到圆形，连接就是扇形，不连接就是弧形
> 
> **drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint)**
> 
> **left、top、right、bottom：**矩形的左上角、右下角的坐标，因为弧形、扇形、圆形都可以是一个矩形内来规划。
> **drawArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean useCenter, Paint paint)**

```java
RectF rect = new RectF();
rect.left = 50;
rect.top = 50;
rect.right = 500;
rect.bottom = 500;
//扇形
canvas.drawArc(rect,0,90,true,paint);
//弧形
canvas.drawArc(rect,120,90,false,paint);
//弧线
paint.setStyle(Paint.Style.STROKE);
canvas.drawArc(rect,230,90,false,paint);
```

**11. 根据path路径绘制图形，更多的是不规则图形**

| Path常用方法|     效果、作用描述|  
| :-------- | :--------| 
|lineTo(x,y)| 若起点为a(10,10),表示将线条从a绘制到终点坐标(x,y)|
|rLineTo(dx,dy)|若起点为a(10,10),表示将线条从a绘制到终点坐标(10+dx,10+dy)|
|moveTo(x,y)|将下一次path绘制的起点坐标移动至(x,y)，比如原本起点在a(10,10)，使用moveTo之后，下次的起点在(x,y)|
|rMoveTo(dx,dy) |将下一次path绘制的起点坐标增加dx、dy，比如原本起点在a(10,10)，使用rMoveTo之后，下次的起点在(10+dx,10+dy)|
|setLastPoint(dx,dy) | 改变前一步path绘制的终点坐标,将终点从(x,y)改变为(dx，dy)|
|addRect | 绘制矩形|
|addRoundRect | 绘制圆角矩形 |
|addCircle |　绘制圆形 | 
|addOval | 绘制椭圆|
|addArc、arcTo|绘制圆弧 |
|close|将path的起点和终点进行闭合|
|Path.Direction.CW|顺时针、clockwise|
|Path.Direction.CCW|逆时针、counter-clockwise|


- **lineTo和rLineTo**
![Alt text](https://i.loli.net/2019/04/14/5cb2bd92f27ed.jpg)

```java
//lineTo和rLineTo
Path path = new Path();
path.lineTo(100, 100);
path.lineTo(500, 500);
canvas.drawPath(path, paint);

Path path2 = new Path();
path2.lineTo(100, 100);
path2.rLineTo(500, 500);
canvas.drawPath(path2, paint);
```


- **rMoveTo和moveTo**
![Alt text](https://i.loli.net/2019/04/14/5cb2c0c5422b8.jpg)

```java
//rMoveTo和moveTo
Path path = new Path();
path.lineTo(100, 100);
path.rMoveTo(100, 200);
path.lineTo(500, 500);
canvas.drawPath(path, paint);

Path path2 = new Path();
path2.lineTo(100, 100);
path2.moveTo(100, 200);
path2.lineTo(500, 500);
canvas.drawPath(path2, paint);
```

- **addRect、addRoundRect、addCircle、addOval**
![Alt text](https://i.loli.net/2019/04/14/5cb2bd935bb05.jpg)
```java
//添加圆形
Path path = new Path();
path.setFillType(Path.FillType.WINDING);
path.addCircle(300,300,200,Path.Direction.CW);
path.addCircle(500,300,200,Path.Direction.CW);
canvas.drawPath(path,paint);
```

- **addArc、arcTo**
都是绘制圆弧的，addArc与arcTo的区别简单来说在于：参数forceMoveTo，是否强制将path的最后一个点移动到圆弧的起点，true表示强制移动，不连接两点，false表示连接两点。addArc相当于默认是true，不连接两点。

```java
//绘制圆弧
addArc(RectF oval, float startAngle, float sweepAngle)
addArc(float left, float top, float right, float bottom, float startAngle,float sweepAngle)

//forceMoveTo：是否强制将path最后一个点移动到圆弧起点，
//true是强制移动，即为不连接两个点；false则连接两个点
arcTo(RectF oval, float startAngle, float sweepAngle,boolean forceMoveTo)
arcTo(RectF oval, float startAngle, float sweepAngle)
arcTo(float left, float top, float right, float bottom, float startAngle,float sweepAngle, boolean forceMoveTo)
```
- **arcTo绘制连接和不连接的两条路径**
![Alt text](https://i.loli.net/2019/04/14/5cb2bd935d391.jpg)

```java
//黄色线条，与圆弧不连接
Path path = new Path();
path.lineTo(100,200);
path.arcTo(300,200,600,500,-90,180,true);
canvas.drawPath(path,paint);

//红色线条，与圆弧连接
paint.setColor(Color.RED);
Path path2 = new Path();
path2.lineTo(200,200);
path2.arcTo(200,200,550,500,-90,180,false);
canvas.drawPath(path2,paint);
```

- **使用Path绘制爱心**
![Alt text](https://i.loli.net/2019/04/14/5cb2c122c72b4.jpg)

```java
//根据path路径绘制一个爱心
Path path = new Path();
path.addArc(200, 200, 400, 400, -225, 225);
path.arcTo(400, 200, 600, 400, -180, 225, false);
path.lineTo(400, 542);
canvas.drawPath(path,paint);
```

**12. Path.setFillType(Path.FillType ft) 设置填充方式**
- EVEN_ODD(交叉填充)
- WINDING （默认值，全部填充）
- INVERSE_EVEN_ODD(反向交叉填充)
- INVERSE_WINDING(反向全部填充)
![Alt text](https://i.loli.net/2019/04/14/5cb2beeb5f45f.jpg)

```java
//完全填充
Path path = new Path();
//绘制区域全部填充
path.setFillType(Path.FillType.WINDING);
//绘制区域重合处不填充
path.setFillType(Path.FillType.EVEN_ODD);
//绘制区域外全部填充
path.setFillType(Path.FillType.INVERSE_WINDING);
//绘制区域重合出填充，绘制区域外全部填充
path.setFillType(Path.FillType.INVERSE_EVEN_ODD);
path.addCircle(300,300,200,Path.Direction.CW);
path.addCircle(500,300,200,Path.Direction.CW);
canvas.drawPath(path,paint);
```
- **效果展示图**

![Alt text](https://i.loli.net/2019/04/14/5cb2bd936f0d1.jpg) 
![Alt text](https://i.loli.net/2019/04/14/5cb2bd937980e.jpg)
- **反向，不在绘制区域内的部分，也会被填充颜色**
![Alt text](https://i.loli.net/2019/04/14/5cb2bf12554e2.jpg)
![Alt text](https://i.loli.net/2019/04/14/5cb2bf125eeb5.jpg)

---

**PDF文件获取地址 : **
[2. 自定义View方法查询手册-Canvas 百度云](https://pan.baidu.com/s/1v2BNdBQfwp7dBrPHykduKQ)  提取码：tkhi



