---
title: 自定义View方法查询手册-Paint
copyright: true
date: 2019-03-07 18:28:18
tags:
- paint
categories:
- Java
---

**系列文章**

[1. 自定义View方法查询手册-Paint](https://www.syncxiao.com/2019/03/07/%E8%87%AA%E5%AE%9A%E4%B9%89View%E6%96%B9%E6%B3%95%E6%9F%A5%E8%AF%A2%E6%89%8B%E5%86%8C-Paint)


[2. 自定义View方法查询手册-Canvas](https://www.syncxiao.com/2019/04/14/%E8%87%AA%E5%AE%9A%E4%B9%89View%E6%96%B9%E6%B3%95%E6%9F%A5%E8%AF%A2%E6%89%8B%E5%86%8C-Canvas)


<!-- more -->

---

## 1. 基本设置

- **为了方便展示，后续操作均基于以下代码进行修改。**

```java
public class HenView6 extends View {
    private String mZHText = "自定义View方法查询手册";

    Paint paint = new Paint();
    Path patn = new Path();

    public HenView6(Context context, AttributeSet attrs) {
        super(context, attrs);
        //设置文字大小
        paint.setTextSize(60);
        //设置线条宽度
        paint.setStrokeWidth(20);

    }

    public HenView6(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }


    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //绘制文字
        canvas.drawText(mZHText,200,200,paint);
        //绘制一个圆
        canvas.drawCircle(500,600,200,paint);
        //绘制一条线段
        canvas.drawLine(100,300,1000,300,paint);
    }
}
```

- **原始效果展示**

![image|center|500x0](https://i.loli.net/2019/02/28/5c7766301b3e5.jpg)

---

## 2. 画笔Paint
| 方法原型 | 方法调用 | 效果展示 |
| --- | --- | :---: |
| 设置透明度和颜色a表示透明度，取值从0-255，从完全透明-不透明.<br>**setARGB(int a, int r, int g, int b)** | 设置透明度为100，颜色为浅水红 <br>paint.setARGB(100,200,0,0)| ![image](https://i.loli.net/2019/02/28/5c777fb0cd291.jpg)|
| 单独设置颜色<br>**setColor(int color)** | `//方式一`<br>paint.setColor(Color.parseColor("#889988"))<br>`//方式二`<br>paint.setColor(Color.GREEN) | ![image](https://i.loli.net/2019/02/28/5c77661fe06f9.jpg)|
| 单独设置透明度<br>**setAlpha(int a)** | 设置透明度为100<br>paint.setAlpha(100) | ![image](https://i.loli.net/2019/02/28/5c777d2d37b20.jpg) |
| 设置文本之间的间距默认值是0，大于0宽松，小于0收紧<br>**setLetterSpacing(float letterSpacing)** | 设置文本之间的间距为0.05<br>paint.setLetterSpacing(0.5f) | ![image](https://i.loli.net/2019/02/28/5c777d4913ff6.jpg)|
| 设置文本对齐方式即文本在原点的左侧、中间、右侧<br>**setTextAlign(Paint.Align align)** | `//左对齐`<br>paint.setTextAlign(Paint.Align.LEFT)<br>`//居中对齐`<br>paint.setTextAlign(Paint.Align.CENTER)<br>`//右对齐`<br>paint.setTextAlign(Paint.Align.RIGHT) | ![image](https://i.loli.net/2019/02/28/5c77b316335a0.jpg) |
| 设置文本区域列表其实就是根据设置的区域显示文本<br>**setTextLocale(Locale locale)** | mZHText = "雨水"<br>`//区域默认是中国`<br>canvas.drawText(mZHText,200,200,paint)<br>`//区域设置为中国台湾`<br>paint.setTextLocale(Locale.TAIWAN)<br>canvas.drawText(mZHText,600,200,paint) | ![image](https://i.loli.net/2019/02/28/5c77b417aa487.jpg) |
| 设置文本横向缩放默认值是1，大于1放宽，小于1锁紧<br>**setTextScaleX(float scaleX)** | 缩放0.5<br>paint.setTextScaleX(0.5f) | ![image](https://i.loli.net/2019/02/28/5c777ebf4f156.jpg) |
| 设置字体类型<br>**setTypeface(Typeface typeface)** | 设置为黑体<br>paint.setTypeface(Typeface.DEFAULT_BOLD) | ![image](https://i.loli.net/2019/02/28/5c777ecd5512c.jpg) |
| 设置字体类型<br>**setTypeface(Typeface typeface)** | 设置为Serif斜体<br>Typeface typeface = Typeface.create<br>(Typeface.SERIF,Typeface.BOLD_ITALIC)<br>paint.setTypeface(typeface) | ![image](https://i.loli.net/2019/02/28/5c777ed91d6c8.jpg) |
| 设置抗锯齿，默认是关闭的<br>**setAntiAlias (boolean aa)** | 开启抗锯齿，注意放大图片观察边缘效果<br>paint.setAntiAlias(true) | ![image](https://i.loli.net/2019/02/28/5c77ac7a9d45f.jpg) |
| 设置绘制模式默认是FILL填充模式，另外还有STROKE画线模式、FILL_AND_STROKE填充画线<br>**setStyle(Paint.Style style)** | 设置为画线模式<br>paint.setStyle(Paint.Style.STROKE) | ![image](https://i.loli.net/2019/02/28/5c77ac9154f10.jpg)|
| 设置线条宽度<br>**setStrokeWidth(float width)** | 设置线条宽度为30<br>paint.setStrokeWidth(30) | ![image](https://i.loli.net/2019/02/28/5c77aca9c896e.jpg) |
| 设置线头形状有三种：默认BUTT 平头、ROUND 圆头、SQUARE 方头<br>**setStrokeCap(Paint.Cap cap)** | 默认平头<br>paint.setStrokeCap(Paint.Cap.BUTT)<br>圆头<br>paint.setStrokeCap(Paint.Cap.ROUND)<br>方头<br>paint.setStrokeCap(Paint.Cap.SQUARE) | ![image](https://i.loli.net/2019/02/28/5c77acb6c64dc.jpg)|
| 设置拐角形状有三种：默认MITER 尖角、BEVEL 平角、ROUND 圆角<br>**setStrokeJoin(Paint.Join join)** | 尖角<br>paint.setStrokeJoin(Paint.Join.MITER)<br>平角paint.setStrokeJoin(Paint.Join.BEVEL)<br>圆角<br>paint.setStrokeJoin(Paint.Join.ROUND) | ![image](https://i.loli.net/2019/02/28/5c77acc154206.jpg)|
| 设置MITER型拐角的延长线的最大值其实就是尖的拐角在超过最大值时，会变成BEVEL型平角，默认值是4,拐角大约是29°的锐角<br>**setStrokeMiter(float miter)** | 设置默认值4<br>paint.setStrokeMiter(4)<br>修改最大值为2<br>paint.setStrokeMiter(2)<br>修改最大值为5<br>paint.setStrokeMiter(5) | ![image](https://i.loli.net/2019/02/28/5c77accb20f0b.jpg) |
| **着色器 Shader** | **paint.setShader(shader))** | |
| **线性渐变LinearGradient** <br>x0 y0 x1 y1：渐变的两个端点的位置 <br>color0 color1：是端点的颜色| **CLAMP模式**<br>Shader shader = <br>new LinearGradient(300, 200, 700, 600, <br>Color.parseColor("#E91E63"), <br>Color.parseColor("#2196F3"), <br>Shader.TileMode.CLAMP) |![16.jpg](https://i.loli.net/2019/03/07/5c8105ae5c2ac.jpg)|  
|| **REPEAT重复模式**<br>Shader shader = <br>new LinearGradient(300, 200, 700, 600, <br>Color.parseColor("#E91E63"), <br>Color.parseColor("#2196F3"), <br>Shader.TileMode.REPEAT) | ![17.jpg](https://i.loli.net/2019/03/07/5c8105f4482bd.jpg)|
| | **MIRROR镜像模式**<br>Shader shader = <br>new LinearGradient(300, 200, 700, 600, <br>Color.parseColor("#E91E63"), <br>Color.parseColor("#2196F3"), <br>Shader.TileMode.MIRROR) |![18.jpg](https://i.loli.net/2019/03/07/5c8105f445367.jpg)|
| **辐射渐变RadialGradient**<br>centerX centerY：辐射中心的坐标 <br>radius：辐射半径 <br>centerColor：辐射中心的颜色 <br>edgeColor：辐射边缘的颜色 | **CLAMP模式**<br>Shader shader = <br>new RadialGradient(500, 420, 100, <br>Color.parseColor("#E91E63"), <br>Color.parseColor("#2196F3"), <br>Shader.TileMode.CLAMP); | ![19.jpg](https://i.loli.net/2019/03/07/5c8105f4613db.jpg)|
| | **REPEAT重复模式**<br>Shader shader = <br>new RadialGradient(500, 420, 100, <br>Color.parseColor("#E91E63"), <br>Color.parseColor("#2196F3"), <br>Shader.TileMode.REPEAT); | ![20.jpg](https://i.loli.net/2019/03/07/5c8105f466f1c.jpg)|
| | **MIRROR镜像模式**<br>Shader shader = <br>new RadialGradient(500, 420, 100, <br>Color.parseColor("#E91E63"), <br>Color.parseColor("#2196F3"), <br>Shader.TileMode.MIRROR); | ![21.jpg](https://i.loli.net/2019/03/07/5c8105f46a81a.jpg)|
| **扫描渐变SweepGradient**<br>cx cy ：扫描的中心 <br>color0：扫描的起始颜色 <br>color1：扫描的终止颜色 | Shader shader = <br>new SweepGradient(500,420,<br>Color.parseColor("#E91E63"),<br>Color.parseColor("#2196F3")); |![22.jpg](https://i.loli.net/2019/03/07/5c8105f468c76.jpg)|
| **BitmapShader 使用bitmap的像素来作为图片或文字的填充**<br>bitmap：用来做模板的Bitmap对象 <br>tileX：横向的 TileMode <br>tileY：纵向的 TileMode | **CLAMP模式**<br>Shader shader = new BitmapShader(bitmap,<br>Shader.TileMode.REPEAT,Shader.TileMode.CLAMP); | ![23.jpg](https://i.loli.net/2019/03/07/5c8106852ddf5.jpg)|
| | **REPEAT重复模式**<br>Shader shader = new BitmapShader(bitmap,<br>Shader.TileMode.REPEAT,Shader.TileMode.REPEAT); |![25.jpg](https://i.loli.net/2019/03/07/5c8106855ab95.jpg) |
| | **MIRROR镜像模式**<br>Shader shader = new BitmapShader(bitmap,<br>Shader.TileMode.REPEAT,Shader.TileMode.MIRROR); | ![24.jpg](https://i.loli.net/2019/03/07/5c81068544d16.jpg)|
| **混合着色器ComposeShader**<br>**`具体请关注下面关于PorterDuff.Mode的介绍`** | `//ShaderA`<br>Shader shaderA = new BitmapShader<br>(bitmapA,Shader.TileMode.REPEAT,<br>Shader.TileMode.CLAMP);<br>`//ShaderB`<br>Shader shaderB = new BitmapShader<br>(bitmapB,Shader.TileMode.REPEAT,<br>Shader.TileMode.CLAMP);<br>`//混合`<br>Shader shader = new ComposeShader<br>(shaderA,shaderB,PorterDuff.Mode.SRC_OVER); | ![image](https://i.loli.net/2019/03/06/5c7f4a2e2cf4d.jpg) |
| **颜色过滤ColorFilter** | **paint.setColorFilter(colorFilter)** ||
| **模拟光照效果<br>LightingColorFilter** | **原样绘制**<br>LightingColorFilter colorFilter = <br>new LightingColorFilter<br>(Color.parseColor("#FFFFFF"),<br>Color.parseColor("#000000")); | ![image](https://i.loli.net/2019/03/06/5c7f4c33b9c14.jpg)|
|| **加深红色**<br>LightingColorFilter colorFilter = <br>new LightingColorFilter<br>(Color.parseColor("#FFFFFF"),<br>Color.parseColor("#FF2222")); | ![image](https://i.loli.net/2019/03/06/5c7f4c40c7ab5.jpg) |
|| **加深绿色**<br>LightingColorFilter colorFilter = <br>new LightingColorFilter<br>(Color.parseColor("#FFFFFF"),<br>Color.parseColor("#22FF22")); | ![image](https://i.loli.net/2019/03/06/5c7f4c4a7676f.jpg) |
|| **加深蓝色**<br>LightingColorFilter colorFilter = <br>new LightingColorFilter<br>(Color.parseColor("#FFFFFF"),<br>Color.parseColor("#2222FF")); | ![image](https://i.loli.net/2019/03/06/5c7f4c542880f.jpg) |
| 使用指定颜色和Mode与绘制对象进行合成<br>**PorterDuffColorFilter**<br>**`具体请关注下面关于PorterDuff.Mode的介绍`** | **合成效果：灰色+圆形空缺**<br>PorterDuffColorFilter colorFilter = <br>new PorterDuffColorFilter<br>(Color.parseColor("#666666"),<br>PorterDuff.Mode.SRC_OUT); | ![image](https://i.loli.net/2019/03/06/5c7f4c5e9064f.jpg)|
| 使用ColorMatrix对颜色进行处理<br>**ColorMatrixColorFilter** | **ColorMatrix是一个4x5的矩阵，<br>通过计算把需要绘制的像素进行转换.<br>其中矩阵中的a、b、c、d四列用于改变颜色配色，<br>e所在列用于改变颜色深浅.** | ![5c7f6c54e89e8](https://i.loli.net/2019/03/06/5c7f6c54e89e8.jpg) |
|| **滑动SeekBar，获取数值，改变红色** <br>`//定义数组`<br>float colorArrays[] = new float[]<br>{red, 0, 0, 0, 0,<br>0, green, 0, 0, 0,<br>0, 0, blue, 0, 0,<br>0, 0, 0, alpha, 0}; <br>`//创建颜色矩阵`<br>ColorMatrix colorMatrix = new ColorMatrix(colorArrays);<br>`//创建颜色过滤器`<br>ColorMatrixColorFilter colorFilter = <br>new ColorMatrixColorFilter(colorMatrix);<br>`//设置颜色过滤器`<br>paint.setColorFilter(colorFilter);    | ![image](https://i.loli.net/2019/03/06/5c7f6df57a398.jpg)|
| | **滑动SeekBar，获取数值，改变绿色，代码同上** | ![image](https://i.loli.net/2019/03/06/5c7f6e03718e2.jpg)|
| | **滑动SeekBar，获取数值，改变蓝色，代码同上** | ![image](https://i.loli.net/2019/03/06/5c7f6e0fc7cbb.jpg) |
| | **滑动SeekBar，获取数值，改变透明度，代码同上** | ![image](https://i.loli.net/2019/03/06/5c7f6e1a7db04.jpg) |
| **修改色调 <br>colorMatrix.setRotate(int axis, float degrees)** <br>第一个参数axis可以使用0、1、2分别表示修改红、绿、蓝色 | **修改绿色色调**<br>colorMatrix.setRotate(1,100);<br>ColorMatrixColorFilter colorFilter = new ColorMatrixColorFilter(colorMatrix);<br>paint.setColorFilter(colorFilter); | ![image](https://i.loli.net/2019/03/06/5c7f7a0dc0ff3.jpg)|
| **修改亮度 <br>colorMatrix.setScale(float rScale, float gScale, float bScale, float aScale)**<br>当亮度按照相同比例修改，图像会变白，如果亮度为0，图像会变黑 | **修改亮度**<br>colorMatrix.setScale(5,9,8,1);<br>ColorMatrixColorFilter colorFilter = new ColorMatrixColorFilter(colorMatrix);<br>paint.setColorFilter(colorFilter); | ![image](https://i.loli.net/2019/03/06/5c7f7a199aab3.jpg)|
| **修改饱和度，默认值1<br>colorMatrix.setSaturation(float sat)**<br>当饱和度为0，图片变成灰色 | **饱和度修改为5<br>**colorMatrix.setSaturation(5);<br>ColorMatrixColorFilter colorFilter = new ColorMatrixColorFilter(colorMatrix);<br>paint.setColorFilter(colorFilter); | ![image](https://i.loli.net/2019/03/06/5c7f7a238c078.jpg) |
| **多个矩阵作用在一起，形成叠加效果** | **叠加效果**<br>ColorMatrix mMatrix=new ColorMatrix();<br>`//合并已有的颜色矩阵`<br>mMatrix.postConcat(colorMatrix);<br>mMatrix.postConcat(colorMatrix2);<br>mMatrix.postConcat(colorMatrix3);<br>`//添加至颜色过滤器`<br>ColorMatrixColorFilter colorFilter = new ColorMatrixColorFilter(mMatrix);<br>`//设置颜色过滤器`<br>paint.setColorFilter(colorFilter); | ![image](https://i.loli.net/2019/03/06/5c7f7a2d04f68.jpg)|
|**Transfer mode 图像传输模式**<br>1.以绘制内容作为源图像;<br>2.以View中已有内容作为目标图像;<br>3.以PorterDuff.Mode作为绘制内容的颜色处理方案.|**paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC))**|![42.jpg](https://i.loli.net/2019/03/07/5c8107971a84f.jpg)|
|**`未开启Off-screen Buffer，绘制出现黑色`**|**注意使用离屏缓冲(Off-screen Buffer),否则绘制会出现黑色**<br>canvas.drawBitmap(bitmap1, 0, 0, paint);<br>paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC));<br>canvas.drawBitmap(bitmap2, 0, 0, paint);|![40.jpg](https://i.loli.net/2019/03/07/5c81079724974.jpg)|
|**`开启Off-screen Buffer，绘制正常`**|`//使用saveLayer开启Off-screen Buffer`<br>int saved = canvas.saveLayer<br>(null,null,Canvas.CLIP_SAVE_FLAG);<br>`//目标图像`<br>canvas.drawBitmap(bitmap1, 0, 0, paint);<br>`//设置Mode.SRC`<br>paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC));<br>`//源图像`<br>canvas.drawBitmap(bitmap2, 0, 0, paint);<br>`//清除Mode`<br>paint.setXfermode(null);<br>`//恢复Off-screen Buffer`<br>canvas.restoreToCount(saved);|![41.jpg](https://i.loli.net/2019/03/07/5c8107972b89c.jpg)|
|**给图形轮廓设置效果 PathEffect **|**paint.setPathEffect(PathEffect effect)**||
|把所有拐角变成圆角<br>**CornerPathEffect(float radius)**|`//参数表示圆角半径`<br>PathEffect pathEffect= new CornerPathEffect(20);<br>paint.setPathEffect(pathEffect);<br>canvas.drawPath(path, paint);|![43.jpg](https://i.loli.net/2019/03/07/5c8106d2ab5c8.jpg)|
|把线条进行随机偏离，使用短线段进行拼接<br>**DiscretePathEffect(float segmentLength, float deviation)**|`//参数分别表示拼接线段的长度、偏移量`<br>PathEffect pathEffect = new DiscretePathEffect(20, 5);<br><br>paint.setPathEffect(pathEffect);<br>canvas.drawPath(path, paint);|![44.jpg](https://i.loli.net/2019/03/07/5c8106d2c16ae.jpg)|
|使用虚线来绘制线条<br>**DashPathEffect(float[] intervals, float phase)**|`//第一个参数是个数组，表示参数格式，必须是偶数，按照[画线长度、空白长度、画线长度、空白长度]排列；第二个参数表示偏移量`<br>PathEffect pathEffect = new DashPathEffect(new float[]{20, 10, 5, 10}, 0);<br>paint.setPathEffect(pathEffect);<br>canvas.drawPath(path, paint);|![45.jpg](https://i.loli.net/2019/03/07/5c8106d2cb9f7.jpg)|
|使用path来绘制虚线<br>**PathDashPathEffect(Path shape, float advance, float phase, PathDashPathEffect.Style style)**|`//shape是用来绘制的path，advance是相邻shape之间的间隔，phase是偏移量，style有TRANSLATE(位移)、ROTATE(旋转)、MORPH(变体)三种`<br>PathEffect pathEffect = new PathDashPathEffect(path, 50, 0, PathDashPathEffect.Style.MORPH);<br>paint.setPathEffect(pathEffect);<br>canvas.drawPath(path, paint);|![46.jpg](https://i.loli.net/2019/03/07/5c8106d2de6eb.jpg)|
|组合效果，按照不同的PathEffect对目标进行绘制<br>**SumPathEffect(PathEffect first, PathEffect second)**|`//参数表示不同的效果，不分先后`<br>PathEffect pathEffect = new SumPathEffect(effect1, effect2);<br>paint.setPathEffect(pathEffect);<br>canvas.drawPath(path, paint);|![47.jpg](https://i.loli.net/2019/03/07/5c8106d2e7a47.jpg)|
|组合效果，先对目标A使用一个PathEffect，目标A变化成目标B，然后再对目标B使用另一个PathEffect<br>**ComposePathEffect(PathEffect outerpe, PathEffect innerpe)**|`//outerpe是后应用的，innerpe是先应用的`<br>PathEffect pathEffect = new ComposePathEffect(effect1, effect2);<br>paint.setPathEffect(pathEffect);<br>canvas.drawPath(path, paint);|![48.jpg](https://i.loli.net/2019/03/07/5c8106d2f159b.jpg)|
|在绘制内容下面增加阴影效果<br>**setShadowLayer(float radius, float dx, float dy, int shadowColor)**|`//参数分别表示阴影模糊范围，x、y方向的阴影偏移量，颜色`<br>paint.setShadowLayer(10, 5, 5, Color.RED);<br>canvas.drawText("Hello HenCoder", 50, 200, paint);|![49.jpg](https://i.loli.net/2019/03/07/5c8106d339f7e.jpg)|
|**在绘制内容上面增加附加效果<br>setMaskFilter(MaskFiltermaskfilter)**|**paint.setMaskFilter(maskFilter)**||
|**模糊效果 <br>BlurMaskFilter(float radius, Blur style)**<br>radius：表示阴影范围<br>style：表示类型|`/NORMAL 内外都模糊绘制/`<br>MaskFilter maskFilter1 = new BlurMaskFilter(50, BlurMaskFilter.Blur.NORMAL);<br>`/INNER 内部模糊，外部不绘制/`<br>MaskFilter maskFilter2 = new BlurMaskFilter(50, BlurMaskFilter.Blur.INNER);<br>`/OUTER 内部不绘制，外部模糊/`<br>MaskFilter maskFilter3 = new BlurMaskFilter(50, BlurMaskFilter.Blur.OUTER);<br>`/SOLID 内部正常绘制，外部模糊/`<br>MaskFilter maskFilter4 = new BlurMaskFilter(50, BlurMaskFilter.Blur.SOLID);|![50.jpg](https://i.loli.net/2019/03/07/5c8106d363c99.jpg)|
|**获取绘制的Path<br>getFillPath(Path src, Path dst)**<br>src：表示绘制图形的原Path<br>dst：表示绘制的实际的Path |右图表示原Path和实际Path的区别，默认情况下原Path和实际Path是一样的，但是如果线条宽度不为0，或者设置了PathEffect，就会存在不同。|![52.jpg](https://i.loli.net/2019/03/07/5c81076c6a814.jpg)|
|**获取绘制文本的Path<br>getTextPath(char[] text, int index, int count,float x, float y, Path path)**|||
|**获取绘制文本的Path<br>getTextPath(String text, int start, int end, float x, float y, Path path)**<br>text：表示需要绘制的文字<br>start：表示从哪个位置开始获取Path<br>end：表示从哪个位置结束获取Path<br> x、y：分别表示绘制时文本原点的x坐标、y坐标<br>path：表示哪个path将获得get到的路径|`//获取文本的Path`<br>paint.getTextPath(text, 0, text.length(), 50, 400, textPath);<br>`//获取文本的Path，长度减少5`<br>paint.getTextPath(text, 0, text.length()-5, 50, 600, textPath2);<br>`//绘制Path，注意不是绘制文本`<br>canvas.drawPath(textPath, pathPaint);|![51.jpg](https://i.loli.net/2019/03/07/5c8106d341911.jpg)|


---
**PDF文件获取地址 : **
[1. 自定义View方法查询手册-Paint 百度云](https://pan.baidu.com/s/1lz5FuN0ne94fk30qzgJF_Q)  提取码：4uh7

[2. 自定义View方法查询手册-Paint 微信扫一扫](https://i.loli.net/2019/03/07/5c81143ee73b2.jpg)
![自定义View方法查询手册-Paint 微信扫一扫](https://i.loli.net/2019/03/07/5c81143ee73b2.jpg)



