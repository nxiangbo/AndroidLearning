# Matrix

| 方法类别   | 相关API                                                | 摘要                                       |
| ---------- | ------------------------------------------------------ | ------------------------------------------ |
| 基本方法   | equals hashCode toString toShortString                 | 比较、 获取哈希值、 转换为字符串           |
| 数值操作   | set reset setValues getValues                          | 设置、 重置、 设置数值、 获取数值          |
| 数值计算   | mapPoints mapRadius mapRect mapVectors                 | 计算变换后的数值                           |
| 设置(set)  | setConcat setRotate setScale setSkew setTranslate      | 设置变换                                   |
| 前乘(pre)  | preConcat preRotate preScale preSkew preTranslate      | 前乘变换                                   |
| 后乘(post) | postConcat postRotate postScale postSkew postTranslate | 后乘变换                                   |
| 特殊方法   | setPolyToPoly setRectToRect rectStaysRect setSinCos    | 一些特殊操作                               |
| 矩阵相关   | invert isAffine(API21) isIdentity                      | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 |



## 构造方法

```java
public Matrix() {} // 构造一个单位矩阵

public Matrix(Matrix src) {} // 将src矩阵拷贝到新的Matrix对象中
```



##  数值操作

#### 1.set

```java
void set (Matrix src)
```

没有返回值，有一个参数，作用是将参数Matrix的数值复制到当前Matrix中。如果参数为空，则重置当前Matrix，相当于`reset()`。

#### 2.reset

```java
void reset ()
```

重置当前Matrix(将当前Matrix重置为单位矩阵)。

#### 3.setValues

```java
void setValues (float[] values)
```

setValues的参数是浮点型的一维数组，长度需要大于9，拷贝数组中的前9位数值赋值给当前Matrix。

#### 4.getValues

```java
void getValues (float[] values)
```

很显然，getValues和setValues是一对方法，参数也是浮点型的一维数组，长度需要大于9，将Matrix中的数值拷贝进参数的前9位中。

### 数值计算

#### 1.mapPoints

计算一组点基于当前Matrix变换后的位置，(由于是计算点，所以参数中的float数组长度一般都是偶数的,若为奇数，则最后一个数值不参与计算)。

```java
void mapPoints (float[] pts)  // pts数组作为参数传递原始数值，计算结果仍存放在pts中

void mapPoints (float[] dst, float[] src) // 计算结果放入dst数组中

void mapPoints (float[] dst, int dstIndex,float[] src, int srcIndex, int pointCount) // 计算部分数值
```

参数：

| 参数       | 摘要                     |
| ---------- | ------------------------ |
| dst        | 目标数据                 |
| dstIndex   | 目标数据存储位置起始下标 |
| src        | 源数据                   |
| srcIndex   | 源数据存储位置起始下标   |
| pointCount | 计算的点个数             |

#### 2.mapRadius

```java
float mapRadius (float radius)
```

测量半径，由于圆可能会因为画布变换变成椭圆，所以此处测量的是平均半径。

#### 3.mapRect

```java
boolean mapRect (RectF rect)

boolean mapRect (RectF dst, RectF src)
```

测量矩形变换后位置。



#### 4.mapVectors

测量向量。

```java
void mapVectors (float[] vecs)

void mapVectors (float[] dst, float[] src)

void mapVectors (float[] dst, int dstIndex, float[] src, int srcIndex, int vectorCount)
```

`mapVectors` 与 `mapPoints` 基本上是相同的，可以直接参照上面的`mapPoints`使用方法。

而两者唯一的区别就是`mapVectors`不会受到位移的影响，这符合向量的定律

### set、pre 与 post

对于四种基本变换 平移(translate)、缩放(scale)、旋转(rotate)、 错切(skew) 它们每一种都三种操作方法，分别为 设置(set)、 前乘(pre) 和 后乘 (post)。而它们的基础是Concat，通过先构造出特殊矩阵然后用原始矩阵Concat特殊矩阵，达到变换的结果。



| 方法 | 简介                                                   |
| ---- | ------------------------------------------------------ |
| set  | 设置，会覆盖掉之前的数值，导致之前的操作失效。         |
| pre  | 前乘，相当于矩阵的右乘， `M' = M * S` (S指为特殊矩阵)  |
| post | 后乘，相当于矩阵的左乘，`M' = S * M` （S指为特殊矩阵） |



### 特殊方法

#### 1.setPolyToPoly

```java
boolean setPolyToPoly (
        float[] src,    // 原始数组 src [x,y]，存储内容为一组点
        int srcIndex,   // 原始数组开始位置
        float[] dst,    // 目标数组 dst [x,y]，存储内容为一组点
        int dstIndex,   // 目标数组开始位置
        int pointCount) // 测控点的数量 取值范围是: 0到4
```



| pointCount | 摘要                                        |
| ---------- | ------------------------------------------- |
| 0          | 相当于`reset`                               |
| 1          | 相当于`translate`                           |
| 2          | 可以进行 缩放、旋转、平移 变换              |
| 3          | 可以进行 缩放、旋转、平移、错切 变换        |
| 4          | 可以进行 缩放、旋转、平移、错切以及任何形变 |

#### 2.setRectToRect

```JAVA
boolean setRectToRect (RectF src,           // 源区域
                RectF dst,                  // 目标区域
                Matrix.ScaleToFit stf)      // 缩放适配模式
```

将源矩形内容填充到目标矩形区域。

ScaleToFit 是一个枚举类型，共包含了四种模式:

| 模式   | 摘要                                           |
| ------ | ---------------------------------------------- |
| CENTER | 居中，对src等比例缩放，将其居中放置在dst中。   |
| START  | 顶部，对src等比例缩放，将其放置在dst的左上角。 |
| END    | 底部，对src等比例缩放，将其放置在dst的右下角。 |
| FILL   | 充满，拉伸src的宽和高，使其完全填充满dst。     |



#### 3.rectStaysRect

判断矩形经过变换后是否仍为矩形，假如Matrix进行了平移、缩放则画布仅仅是位置和大小改变，矩形变换后仍然为矩形，但Matrix进行了非90度倍数的旋转或者错切，则矩形变换后就不再是矩形了



#### 4.setSinCos

设置sinCos值，这个是控制Matrix旋转的，由于Matrix已经封装好了Rotate方法，所以这个并不常用，在此仅作概述。

```java
// 方法一
void setSinCos (float sinValue,     // 旋转角度的sin值
                float cosValue)     // 旋转角度的cos值

// 方法二
void setSinCos (float sinValue,     // 旋转角度的sin值
                float cosValue,     // 旋转角度的cos值
                float px,           // 中心位置x坐标
                float py)           // 中心位置y坐标
```

### 矩阵相关

| 方法       | 摘要                                                 |
| ---------- | ---------------------------------------------------- |
| invert     | 求矩阵的逆矩阵                                       |
| isAffine   | 判断当前矩阵是否为仿射矩阵，API21(5.0)才添加的方法。 |
| isIdentity | 判断当前矩阵是否为单位矩阵。                         |



>**仿射变换**，又称**仿射映射**，是指在[几何](https://zh.wikipedia.org/wiki/%E5%87%A0%E4%BD%95)中，一个[向量空间](https://zh.wikipedia.org/wiki/%E5%90%91%E9%87%8F%E7%A9%BA%E9%97%B4)进行一次[线性变换](https://zh.wikipedia.org/wiki/%E7%BA%BF%E6%80%A7%E5%8F%98%E6%8D%A2)并接上一个[平移](https://zh.wikipedia.org/wiki/%E5%B9%B3%E7%A7%BB)，变换为另一个向量空间。
>
>一个对向量$\vec{x}$平移$\vec{b}$，与旋转放大缩小$A​$的仿射映射为
>$$
>{\vec{y}=A\vec{x}+\vec{b}}
>$$
>
>
>上式在[齐次坐标](https://zh.wikipedia.org/wiki/%E9%BD%90%E6%AC%A1%E5%9D%90%E6%A0%87)上，等价于下面的式子
>$$
>\begin{bmatrix}{\vec{y} \\ 1} \end{bmatrix} = \begin{bmatrix}A & \vec{b} \\ 0,\cdots{0} & 1 \\ \end{bmatrix} \begin{bmatrix}{\vec{x} \\ 1} \end{bmatrix}
>$$





