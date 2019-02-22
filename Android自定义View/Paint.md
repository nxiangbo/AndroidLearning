# Paint

Paint 类似于画笔的功能，可以通过Paint设置绘制View的颜色，效果以及style。

## 绘制颜色



| 方法                         | 功能                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| setColor、setARGB、setShader | 设置颜色                                                     |
| setColorFilter               | 为绘制设置颜色过滤                                           |
| setXfermode                  | 以绘制的内容作为源图像，以 View 中已有的内容作为目标图像，选取一个 `PorterDuff.Mode` 作为绘制内容的颜色处理方案 |
|                              |                                                              |

### setShader



setColor、setARGB方法使用比较简单，直接设置颜色即可。这里着重说一下setShader.

```java
public Shader setShader(Shader shader) {}
```

在 Android 的绘制里使用 `Shader` ，并不直接用 `Shader` 这个类，而是用它的几个子类。具体来讲有 `LinearGradient` `RadialGradient` `SweepGradient` `BitmapShader` `ComposeShader` 



###  setColorFilter

在 `Paint` 里设置 `ColorFilter` ，使用的是 `Paint.setColorFilter(ColorFilter filter)` 方法。 `ColorFilter` 并不直接使用，而是使用它的子类。它共有三个子类：`LightingColorFilter``PorterDuffColorFilter` 和 `ColorMatrixColorFilter`。

```java
 public ColorFilter setColorFilter(ColorFilter filter) {}
```



### setXfermode

以绘制的内容作为源图像，以 View 中已有的内容作为目标图像，选取一个 `PorterDuff.Mode` 作为绘制内容的颜色处理方案。



#### PorterDuff.Mode

`PorterDuff.Mode` 是用来指定两个图像共同绘制时的颜色策略的。它是一个 enum，不同的 `Mode` 可以指定不同的策略。

PorterDuff 使用方式：

![](images/porterduff.jpg)

```java
Paint paint = new Paint();
canvas.drawBitmap(destinationImage, 0, 0, paint);

PorterDuff.Mode mode = // choose a mode
paint.setXfermode(new PorterDuffXfermode(mode));

canvas.drawBitmap(sourceImage, 0, 0, paint);
```

具体来说， `PorterDuff.Mode` 一共有 17 个，可以分为两类：

1. Alpha 合成 (Alpha Compositing)
2. 混合 (Blending)

第一类，Alpha 合成，其实就是 「PorterDuff」 这个词所指代的算法。 「PorterDuff」 并不是一个具有实际意义的词组，而是两个人的名字（准确讲是姓）。这两个人当年共同发表了一篇论文，描述了 12 种将两个图像共同绘制的操作（即算法）。而这篇论文所论述的操作，都是关于 Alpha 通道（也就是我们通俗理解的「透明度」）的计算的，后来人们就把这类计算称为**Alpha 合成** ( Alpha Compositing ) 。

![](images/portduff.jpg)

第二类，混合，也就是 Photoshop 等制图软件里都有的那些混合模式（`multiply` `darken` `lighten`之类的）。这一类操作的是颜色本身而不是 `Alpha` 通道，并不属于 `Alpha` 合成，所以和 Porter 与 Duff 这两个人也没什么关系，不过为了使用的方便，它们同样也被 Google 加进了 `PorterDuff.Mode`里。

![](images/porterduff02.jpg)

## 绘制效果

| 方法           | 功能                                |
| -------------- | ----------------------------------- |
| setAntiAlias   | 设置锯齿效果                        |
| setStyle       | 设置图形是线条风格还是填充风格      |



### 线条形状



| 方法           | 功能                                |
| -------------- | ----------------------------------- |
| setStrokeWidth | 设置线条宽度                        |
| setStrokeCap   | 设置线头形状                        |
| setStrokeJoin  | 设置拐角的形状                      |
| setStrokeMiter | 设置 `MITER` 型拐角的延长线的最大值 |



### 色彩优化

`Paint` 的色彩优化有两个方法： `setDither(boolean dither)` 和 `setFilterBitmap(boolean filter)`。它们的作用都是让画面颜色变得更加「顺眼」，但原理和使用场景是不同的。

setDither 设置抖动

setFilterBitmap 设置是否使用双线性过滤来绘制 `Bitmap`

### setPathEffect(PathEffect effect)

使用 `PathEffect` 来给图形的轮廓设置效果。对 `Canvas` 所有的图形绘制有效，也就是 `drawLine()``drawCircle()` `drawPath()` 这些方法。



`PathEffect` 分为两类，单一效果的 `CornerPathEffect` `DiscretePathEffect` `DashPathEffect` `PathDashPathEffect` ，和组合效果的 `SumPathEffect` `ComposePathEffect`。



### setShadowLayer(float radius, float dx, float dy, int shadowColor)

