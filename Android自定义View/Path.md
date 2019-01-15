# Path 



> The Path class encapsulates compound (multiple contour) geometric paths
> consisting of straight line segments, quadratic curves, and cubic curves.
> It can be drawn with canvas.drawPath(path, paint), either filled or stroked
> (based on the paint's Style), or it can be used for clipping or to draw
> text on a path.

Path类封装了由直线段，二次曲线和三次曲线组成的复合（多个轮廓）几何路径。 它可以使用canvas.drawPath（path，paint）绘制，填充或描边（基于绘制的样式），或者它可以用于剪切或在路径上绘制文本。

在自定义View的过程中，Path是非常有用的。通过它可以绘制任何直线，贝塞尔曲线，以及添加圆形或者矩形等。

| 作用            | 相关方法                                                     | 备注                                                         |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 移动起点        | moveTo                                                       | 移动下一次操作的起点位置                                     |
| 设置终点        | setLastPoint                                                 | 重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同 |
| 连接直线        | lineTo                                                       | 添加上一个点到当前点之间的直线到Path                         |
| 闭合路径        | close                                                        | 连接第一个点连接到最后一个点，形成一个闭合区域               |
| 添加内容        | addRect, addRoundRect, addOval, addCircle, addPath, addArc, arcTo | 添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别) |
| 是否为空        | isEmpty                                                      | 判断Path是否为空                                             |
| 是否为矩形      | isRect                                                       | 判断path是否是一个矩形                                       |
| 替换路径        | set                                                          | 用新的路径替换到当前路径所有内容                             |
| 偏移路径        | offset                                                       | 对当前路径之前的操作进行偏移(不会影响之后的操作)             |
| 贝塞尔曲线      | quadTo, cubicTo                                              | 分别为二次和三次贝塞尔曲线的方法                             |
| rXxx方法        | rMoveTo, rLineTo, rQuadTo, rCubicTo                          | **不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)** |
| 填充模式        | setFillType, getFillType, isInverseFillType, toggleInverseFillType | 设置,获取,判断和切换填充模式                                 |
| 提示方法        | incReserve                                                   | 提示Path还有多少个点等待加入**(这个方法貌似会让Path优化存储结构)** |
| 布尔操作(API19) | op                                                           | 对两个Path进行布尔运算(即取交集、并集等操作)                 |
| 计算边界        | computeBounds                                                | 计算Path的边界                                               |
| 重置路径        | reset, rewind                                                | 清除Path中的内容 **reset不保留内部数据结构，但会保留FillType.** **rewind会保留内部的数据结构，但不保留FillType** |
| 矩阵操作        | transform                                                    | 矩阵变换                                                     |





### moveTo、 setLastPoint、 lineTo 和 close

由于Path的有些知识点无法单独来讲，所以本次采取了一次讲一组方法。

按照惯例，先创建画笔：

```java
Paint mPaint = new Paint();             // 创建画笔
mPaint.setColor(Color.BLACK);           // 画笔颜色 - 黑色
mPaint.setStyle(Paint.Style.STROKE);    // 填充模式 - 描边
mPaint.setStrokeWidth(10);              // 边框宽度 - 10
```

#### lineTo：

```java
public void lineTo (float x, float y)
```

**lineTo是指从某个点到参数坐标点之间连一条线**，某个点是指**上次操作结束的点**，若没有进行过操作，默认是**原点**。



#### moveTo 和 setLastPoint：

方法预览：

```
// moveTo
public void moveTo (float x, float y)

// setLastPoint
public void setLastPoint (float dx, float dy)
```

这两个方法虽然在作用上有相似之处，但实际上却是完全不同的两个东东，具体参照下表：

| 方法名       | 简介                         | 是否影响之前的操作 | 是否影响之后操作 |
| ------------ | ---------------------------- | ------------------ | ---------------- |
| moveTo       | 移动下一次操作的起点位置     | 否                 | 是               |
| setLastPoint | 设置之前操作的最后一个点位置 | 是                 | 是               |



#### close

方法预览：

```
public void close ()
```

close方法用于连接当前最后一个点和最初的一个点(如果两个点不重合的话)，最终形成一个封闭的图形。

### addXxx与arcTo

这次内容主要是在Path中添加基本图形，重点区分addArc与arcTo。

#### 第一类(基本形状)

方法预览：

```java
// 第一类(基本形状)

// 圆形
public void addCircle (float x, float y, float radius, Path.Direction dir)
// 椭圆
public void addOval (RectF oval, Path.Direction dir)
// 矩形
public void addRect (float left, float top, float right, float bottom, Path.Direction dir)
public void addRect (RectF rect, Path.Direction dir)
// 圆角矩形
public void addRoundRect (RectF rect, float[] radii, Path.Direction dir)
public void addRoundRect (RectF rect, float rx, float ry, Path.Direction dir)
```

**这一类就是在path中添加一个基本形状，基本形状部分和前面所讲的绘制基本形状并无太大差别，详情参考Canvas之绘制图形, 本次只将其中不同的部分摘出来详细讲解一下。**

上面方法**在最后都有一个 Path.Direction**。

Direction的意思是 方向，趋势。 点进去看一下会发现Direction是一个枚举(Enum)类型，里面只有两个枚举常量，如下：

| 类型 | 解释              | 翻译   |
| ---- | ----------------- | ------ |
| CW   | clockwise         | 顺时针 |
| CCW  | counter-clockwise | 逆时针 |





#### 第二类(Path)

方法预览：

```
// 第二类(Path)
// path
public void addPath (Path src)
public void addPath (Path src, float dx, float dy)
public void addPath (Path src, Matrix matrix)
```

这个相对比较简单，也很容易理解，就是将两个Path合并成为一个。

第三个方法是将src添加到当前path之前先使用Matrix进行变换。

第二个方法比第一个方法多出来的两个参数是将src进行了位移之后再添加进当前path中。



#### 第三类(addArc与arcTo)

方法预览：

```java
// 第三类(addArc与arcTo)

// addArc
public void addArc (RectF oval, float startAngle, float sweepAngle)
// arcTo
public void arcTo (RectF oval, float startAngle, float sweepAngle)
public void arcTo (RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo)
```

参数表：

| 参数        | 摘要                              |
| ----------- | --------------------------------- |
| oval        | 圆弧的外切矩形。                  |
| startAngle  | 开始角度                          |
| sweepAngle  | 扫过角度(-360 <= sweepAngle <360) |
| forceMoveTo | 是否强制使用MoveTo                |



addArc与arcTo区别

| 名称   | 作用               | 区别                                                         |
| ------ | ------------------ | ------------------------------------------------------------ |
| addArc | 添加一个圆弧到path | 直接添加一个圆弧到path中                                     |
| arcTo  | 添加一个圆弧到path | 添加一个圆弧到path，如果圆弧的起点和上次最后一个坐标点不相同，就连接两个点 |



### 第3组：isEmpty、 isRect、isConvex、 set 和 offset

这一组比较简单，稍微说一下就可以了。

#### isEmpty

方法预览：

```
public boolean isEmpty ()
```

判断path中是否包含内容。

#### isRect

方法预览：

```java
public boolean isRect (RectF rect)
```

判断path是否是一个矩形，如果是一个矩形的话，会将矩形的信息存放进参数rect中。

#### set

方法预览：

```
public void set (Path src)
```

将新的path赋值到现有path。

#### offset

方法预览：

```
public void offset (float dx, float dy)
public void offset (float dx, float dy, Path dst)
```

这个的作用也很简单，就是对path进行一段平移，它和Canvas中的translate作用很像，但Canvas作用于整个画布，而path的offset只作用于当前path。



## rXXX方法

**rXxx方法的坐标使用的是相对位置(基于当前点的位移)，而之前方法的坐标是绝对位置(基于当前坐标系的坐标)。**



## 填充模式

机器判断图形内外，一般有以下两种方法：

> PS：此处所有的图形均为封闭图形，不包括图形不封闭这种情况。

| 方法           | 判定条件                                  | 解释                                                         |
| -------------- | ----------------------------------------- | ------------------------------------------------------------ |
| 奇偶规则       | 奇数表示在图形内，偶数表示在图形外        | 从任意位置p作一条射线， 若与该射线相交的图形边的数目为奇数，则p是图形内部点，否则是外部点。 |
| 非零环绕数规则 | 若环绕数为0表示在图形外，非零表示在图形内 | 首先使图形的边变为矢量。将环绕数初始化为零。再从任意位置p作一条射线。当从p点沿射线方向移动时，对在每个方向上穿过射线的边计数，每当图形的边从右到左穿过射线时，环绕数加1，从左到右时，环绕数减1。处理完图形的所有相关边之后，若环绕数为非零，则p为内部点，否则，p是外部点。 |



### 奇偶规则

![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/path-tianch01.jpg)

P1: 从P1发出一条射线，发现图形与该射线相交边数为0，偶数，故P1点在图形外部。
P2: 从P2发出一条射线，发现图形与该射线相交边数为1，奇数，故P2点在图形内部。
P3: 从P3发出一条射线，发现图形与该射线相交边数为2，偶数，故P3点在图形外部。

### 非零环绕数规则

![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/path02.jpg)

P1: 从P1点发出一条射线，沿射线防线移动，并没有与边相交点部分，环绕数为0，故P1在图形外边。
P2: 从P2点发出一条射线，沿射线方向移动，与图形点左侧边相交，该边从左到右穿过穿过射线，环绕数－1，最终环绕数为－1，故P2在图形内部。
P3: 从P3点发出一条射线，沿射线方向移动，在第一个交点处，底边从右到左穿过射线，环绕数＋1，在第二个交点处，右侧边从左到右穿过射线，环绕数－1，最终环绕数为0，故P3在图形外部。



### Android中的填充模式

Android中的填充模式有四种，是封装在Path中的一个枚举。

| 模式             | 简介             |
| ---------------- | ---------------- |
| EVEN_ODD         | 奇偶规则         |
| INVERSE_EVEN_ODD | 反奇偶规则       |
| WINDING          | 非零环绕数规则   |
| INVERSE_WINDING  | 反非零环绕数规则 |



### Android与填充模式相关的方法

> 这些都是Path中的方法。

| 方法                  | 作用                                           |
| --------------------- | ---------------------------------------------- |
| setFillType           | 设置填充规则                                   |
| getFillType           | 获取当前填充规则                               |
| isInverseFillType     | 判断是否是反向(INVERSE)规则                    |
| toggleInverseFillType | 切换填充规则(即原有规则与反向规则之间相互切换) |



## 布尔操作

**布尔操作是两个Path之间的运算，主要作用是用一些简单的图形通过一些规则合成一些相对比较复杂，或难以直接得到的图形**。

Path布尔运算逻辑如下

| 逻辑名称           | 类比 | 说明                                   | 示意图                                                       |
| ------------------ | ---- | -------------------------------------- | ------------------------------------------------------------ |
| DIFFERENCE         | 差集 | Path1中减去Path2后剩下的部分           | ![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/path_diff.jpg) |
| REVERSE_DIFFERENCE | 差集 | Path2中减去Path1后剩下的部分           | ![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/path_reverse_diff.jpg) |
| INTERSECT          | 交集 | Path1与Path2相交的部分                 | ![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/path_in.jpg) |
| UNION              | 并集 | 包含全部Path1和Path2                   | ![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/path_union.jpg) |
| XOR                | 异或 | 包含Path1与Path2但不包括两者相交的部分 | ![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/path_xor.jpg) |

在Path中的布尔运算有两个方法

```java
boolean op (Path path, Path.Op op)
boolean op (Path path1, Path path2, Path.Op op)
```

两个方法中的返回值用于判断布尔运算是否成功，它们使用方法如下:

```java
// 对 path1 和 path2 执行布尔运算，运算方式由第二个参数指定，运算结果存入到path1中。
path1.op(path2, Path.Op.DIFFERENCE);

// 对 path1 和 path2 执行布尔运算，运算方式由第三个参数指定，运算结果存入到path3中。
path3.op(path1, path2, Path.Op.DIFFERENCE)
```



## 计算边界

这个方法主要作用是计算Path所占用的空间以及所在位置,方法如下：

```
void computeBounds (RectF bounds, boolean exact)
```

它有两个参数：

| 参数   | 作用                                                       |
| ------ | ---------------------------------------------------------- |
| bounds | 测量结果会放入这个矩形                                     |
| exact  | 是否精确测量，目前这一个参数作用已经废弃，一般写true即可。 |



## 重置路径

重置Path有两个方法，分别是reset和rewind，两者区别主要有一下两点：

| 方法   | 是否保留FillType设置 | 是否保留原有数据结构 |
| ------ | -------------------- | -------------------- |
| reset  | 是                   | 否                   |
| rewind | 否                   | 是                   |

**这个两个方法应该何时选择呢？**

选择权重: FillType > 数据结构

*因为“FillType”影响的是显示效果，而“数据结构”影响的是重建速度*