# Camera



> A camera instance can be used to compute **3D transformations** and generate a matrix that can be applied, for instance, on a {@link Canvas}.



## Camera常用方法表

| 方法类别 | 相关API                                                      | 简介                   |
| -------- | ------------------------------------------------------------ | ---------------------- |
| 基本方法 | save、restore                                                | 保存、 回滚            |
| 常用方法 | getMatrix、applyToCanvas                                     | 获取Matrix、应用到画布 |
| 平移     | translate                                                    | 位移                   |
| 旋转     | rotat (API 12)、rotateX、rotateY、rotateZ                    | 各种旋转               |
| 相机位置 | setLocation (API 12)、getLocationX (API 16)、getLocationY (API 16)、getLocationZ (API 16) | 设置与获取相机位置     |



### 3D坐标系

我们Camera使用的3维坐标系是**左手坐标系，即左手手臂指向x轴正方向，四指弯曲指向y轴正方向，此时展开大拇指指向的方向是z轴正方向**。



**2D 和 3D 坐标是通过Matrix关联起来的，所以你可以认为两者是同一个坐标系，但又有差别，重点就是y轴方向不同。**

| 坐标系       | 2D坐标系 | 3D坐标系     |
| ------------ | -------- | ------------ |
| 原点默认位置 | 左上角   | 左上角       |
| X 轴默认方向 | 右       | 右           |
| Y 轴默认方向 | 下       | 上           |
| Z 轴默认方向 | 无       | 垂直屏幕向内 |





## 基本方法

基本方法就有两个`save` 和`restore`，主要作用为`保存当前状态和恢复到上一次保存的状态`，通常成对使用，常用格式如下:

```java
camera.save();		// 保存状态
... 			// 具体操作
camera.retore();	// 回滚状态
```



## 常用方法

这两个方法是Camera中最基础也是最常用的方法。

### getMatrix

```
void getMatrix (Matrix matrix)
```

计算当前状态下矩阵对应的状态，并将计算后的矩阵赋值给参数matrix。

### applyToCanvas

```
void applyToCanvas (Canvas canvas)
```

计算当前状态下单矩阵对应的状态，并将计算后的矩阵应用到指定的canvas上。



## 平移

> **声明：以下示例中 Matrix 的平移均使用 postTranslate 来演示，实际情况中使用set、pre 或 post 需要视情况而定。**

```
void translate (float x, float y, float z)
```

和2D平移类似，只不过是多出来了一个维度，从只能在2D平面上平移到在3D空间内平移，不过，此处仍有几个要点需要重点对待。

### 沿x轴平移

```
camera.translate(x, 0, 0);

matrix.postTranslate(x, 0);
```

两者x轴同向，所以 Camera 和 Matrix 在沿x轴平移上是一致的。



### 沿y轴平移

这个就有点意思了，两个坐标系相互关联，但是两者的y轴方向是相反的，很容易把人搞迷糊。你可以这么玩:

```
Camera camera = new Camera();
camera.translate(0, 100, 0);    // camera - 沿y轴正方向平移100像素

Matrix matrix = new Matrix();
camera.getMatrix(matrix);
matrix.postTranslate(0,100);    // matrix - 沿y轴正方向平移100像素

Log.i(TAG, "Matrix: "+matrix.toShortString());
```

在上面这种写法，虽然用了5行代码，但是效果却和 `Matrix matrix = new Matrix();` 一样



### 沿z轴平移

这个不仅有趣，还容易蒙逼，上面两种情况再怎么闹腾也只是在2D平面上，而z轴的出现则让其有了空间感。

**当View和摄像机在同一条直线上时:** 此时沿z轴平移相当于缩放的效果，缩放中心为摄像机所在(x, y)坐标，当View接近摄像机时，看起来会变大，远离摄像机时，看起来会变小，**近大远小**。

**当View和摄像机不在同一条直线上时:** 当View远离摄像机的时候，View在缩小的同时也在不断接近摄像机在屏幕投影位置(通常情况下为Z轴，在平面上表现为接近坐标原点)。相反，当View接近摄像机的时候，View在放大的同时会远离摄像机在屏幕投影位置。



## 旋转

旋转是Camera制作3D效果的核心，不过它制作出来的并不能算是真正的3D，而是伪3D，因为View是没有厚度的。

```java
// (API 12) 可以控制View同时绕x，y，z轴旋转，可以由下面几种方法复合而来。
void rotate (float x, float y, float z);

// 控制View绕单个坐标轴旋转
void rotateX (float deg);
void rotateY (float deg);
void rotateZ (float deg);
```



### 旋转中心

旋转中心默认是坐标原点，对于图片来说就是左上角位置。

我们都知道，在2D中，不论是旋转，错切还是缩放都是能够指定操作中心点位置的，但是在3D中却没有默认的方法，如果我们想要让图片围绕中心点旋转怎么办? 这就要使用到我们在[Matrix原理](http://www.gcssloop.com/customview/Matrix_Basic)提到过的方法:

```java
Matrix temp = new Matrix();		// 临时Matrix变量
this.getMatrix(temp);			// 获取Matrix
temp.preTranslate(-centerX, -centerY);	// 使用pre将旋转中心移动到和Camera位置相同。
temp.postTranslate(centerX, centerY);	// 使用post将图片(View)移动到原来的位置
```