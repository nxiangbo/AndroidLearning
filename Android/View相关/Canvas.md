# Canvas



> The Canvas class holds the "draw" calls. To draw something, you need
4 basic components: A Bitmap to hold the pixels, a Canvas to host
the draw calls (writing into the bitmap), a drawing primitive (e.g. Rect,
Path, text, Bitmap), and a paint (to describe the colors and styles for the
drawing).

## 绘制基本图形

Canvas 主要提供了在画布上绘制的功能。它有一系列的drawXXX()方法。可以绘制颜色，绘制基本图形，绘制文字,绘制贝塞尔曲线等等。

|              |                                                              |                                                              |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 绘制颜色     | drawColor, drawRGB, drawARGB                                 | 使用单一颜色填充整个画布                                     |
| 绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧                |
| 绘制图片     | drawBitmap, drawPicture                                      | 绘制位图和图片                                               |
| 绘制文本     | drawText, drawPosText, drawTextOnPath                        | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字 |
| 绘制路径     | drawPath                                                     | 绘制路径，绘制贝塞尔曲线时也需要用到该函数                   |
| 顶点操作     | drawVertices, drawBitmapMesh                                 | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用 |

## 裁切

当然，Canvas的功能不止于此，它还有裁切指定范围的功能，对应的是clipXXX方法。

| clipRect | 设置画布的显示区域 |
| -------- | ------------------ |
| clipPath |                    |





## 画布变换

它还具有几何变换的功能。平移，旋转，缩放，也可以使用Matrix自定义变换规则，setMatrix(Matrix matrix) concat(Matrix matrix)。

| 画布变换     | translate, scale, rotate, skew | 依次为 位移、缩放、 旋转、错切                               |
| ------------ | ------------------------------ | ------------------------------------------------------------ |
| Matrix(矩阵) | getMatrix, setMatrix, concat   | 实际上画布的位移，缩放等操作的都是图像矩阵Matrix， 只不过Matrix比较难以理解和使用，故封装了一些常用的方法。 |



### 平移

translate是坐标系的移动，可以为图形绘制选择一个合适的坐标系。 **请注意，位移是基于当前位置移动，而不是每次基于屏幕左上角的(0,0)点移动**

```java
// 在坐标原点绘制一个黑色圆形
mPaint.setColor(Color.BLACK);
canvas.translate(200,200);
canvas.drawCircle(0,0,100,mPaint);

// 在坐标原点绘制一个蓝色圆形
mPaint.setColor(Color.BLUE);
canvas.translate(200,200);
canvas.drawCircle(0,0,100,mPaint);
```





### 缩放

缩放提供了两个方法，如下：

```java
public void scale (float sx, float sy)

public final void scale (float sx, float sy, float px, float py) // 后两个参数用于控制缩放中心位置
```



缩放比例(sx,sy)取值范围详解：

| 取值范围(n) | 说明                                           |
| ----------- | ---------------------------------------------- |
| (-∞, -1)    | 先根据缩放中心放大n倍，再根据中心轴进行翻转    |
| -1          | 根据缩放中心轴进行翻转                         |
| (-1, 0)     | 先根据缩放中心缩小到n，再根据中心轴进行翻转    |
| 0           | 不会显示，若sx为0，则宽度为0，不会显示，sy同理 |
| (0, 1)      | 根据缩放中心缩小到n                            |
| 1           | 没有变化                                       |
| (1, +∞)     | 根据缩放中心放大n倍                            |



### 旋转

旋转提供了两种方法：

```java
public void rotate (float degrees)

public final void rotate (float degrees, float px, float py)
```

默认的旋转中心依旧是坐标原点。

###  错切

skew这里翻译为错切，错切是特殊类型的线性变换。

错切只提供了一种方法：

```java
public void skew (float sx, float sy)
```

**参数含义：float sx:将画布在x方向上倾斜相应的角度，sx倾斜角度的tan值，float sy:将画布在y轴方向上倾斜相应的角度，sy为倾斜角度的tan值.**



## 画布快照和回滚

| 方法           | 功能                   |
| -------------- | ---------------------- |
| save           | 保存当前状态           |
| restore        | 回滚到上一次保存的状态 |
| saveLayerXxx   | 保存图层状态           |
| restoreToCount | 回滚到指定状态         |
| getSaveCount   | 获取保存次数           |

类似于Photoshop，Canvas也有图层的概念。当绘制比较复杂的view时，可能需要绘制多个图层。而有一个栈是专门用于存放图层的，当调用save方法时，将当前图层状态压入到栈中。当调用restore方法时，则将栈中的栈顶图层出栈。通常情况下，save方法和restore方法都是成对使用。

> 注意：restore和save必须成对出现，如果restore出现次数比save多，则会报错。错误信息为`java.lang.IllegalStateException: Underflow in restore - more restores than saves`

save有两个方法

```java
// 保存全部状态
public int save ()

// 根据saveFlags参数保存一部分状态
public int save (int saveFlags)
```



##### SaveFlags

| 名称                       | 简介                                            |
| -------------------------- | ----------------------------------------------- |
| ALL_SAVE_FLAG              | 默认，保存全部状态                              |
| CLIP_SAVE_FLAG             | 保存剪辑区                                      |
| CLIP_TO_LAYER_SAVE_FLAG    | 剪裁区作为图层保存                              |
| FULL_COLOR_LAYER_SAVE_FLAG | 保存图层的全部色彩通道                          |
| HAS_ALPHA_LAYER_SAVE_FLAG  | 保存图层的alpha(不透明度)通道                   |
| MATRIX_SAVE_FLAG           | 保存Matrix信息( translate, rotate, scale, skew) |



#### saveLayerXxx

saveLayerXxx有比较多的方法：

```java
// 无图层alpha(不透明度)通道
public int saveLayer (RectF bounds, Paint paint)
public int saveLayer (RectF bounds, Paint paint, int saveFlags)
public int saveLayer (float left, float top, float right, float bottom, Paint paint)
public int saveLayer (float left, float top, float right, float bottom, Paint paint, int saveFlags)

// 有图层alpha(不透明度)通道
public int saveLayerAlpha (RectF bounds, int alpha)
public int saveLayerAlpha (RectF bounds, int alpha, int saveFlags)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha, int saveFlags)
```

**注意：saveLayerXxx方法会让你花费更多的时间去渲染图像(图层多了相互之间叠加会导致计算量成倍增长)，使用前请谨慎，如果可能，尽量避免使用。**

使用saveLayerXxx方法，也会将图层状态也放入状态栈中，同样使用restore方法进行恢复。



