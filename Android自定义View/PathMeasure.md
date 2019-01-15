# PathMeasure

### 构造方法

| 方法名                                      | 释义                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| PathMeasure()                               | 创建一个空的PathMeasure                                      |
| PathMeasure(Path path, boolean forceClosed) | 创建 PathMeasure 并关联一个指定的Path(Path需要已经创建完成)。 |

- 不论 forceClosed 设置为何种状态(true 或者 false)， 都不会影响原有Path的状态，**即 Path 与 PathMeasure 关联之后，之前的的 Path 不会有任何改变。**
- forceClosed 的设置状态可能会影响测量结果，**如果 Path 未闭合但在与 PathMeasure 关联的时候设置 forceClosed 为 true 时，测量结果可能会比 Path 实际长度稍长一点，获取到到是该 Path 闭合时的状态。**

### 公共方法

| 返回值  | 方法名                                                       | 释义                               |
| ------- | ------------------------------------------------------------ | ---------------------------------- |
| void    | setPath(Path path, boolean forceClosed)                      | 关联一个Path                       |
| boolean | isClosed()                                                   | 是否闭合                           |
| float   | getLength()                                                  | 获取Path的长度                     |
| boolean | nextContour()                                                | 跳转到下一个轮廓                   |
| boolean | getSegment(float startD, float stopD, Path dst, boolean startWithMoveTo) | 截取片段                           |
| boolean | getPosTan(float distance, float[] pos, float[] tan)          | 获取指定长度的位置坐标及该点切线值 |
| boolean | getMatrix(float distance, Matrix matrix, int flags)          | 获取指定长度的位置坐标及该点Matrix |



### getSegment

getSegment 用于获取Path的一个片段，方法如下：

```java
boolean getSegment (float startD, float stopD, Path dst, boolean startWithMoveTo)
```

方法各个参数释义：

| 参数            | 作用                             | 备注                                                         |
| --------------- | -------------------------------- | ------------------------------------------------------------ |
| 返回值(boolean) | 判断截取是否成功                 | true 表示截取成功，结果存入dst中，false 截取失败，不会改变dst中内容 |
| startD          | 开始截取位置距离 Path 起点的长度 | 取值范围: 0 <= startD < stopD <= Path总长度                  |
| stopD           | 结束截取位置距离 Path 起点的长度 | 取值范围: 0 <= startD < stopD <= Path总长度                  |
| dst             | 截取的 Path 将会添加到 dst 中    | 注意: 是添加，而不是替换                                     |
| startWithMoveTo | 起始点是否使用 moveTo            | 用于保证截取的 Path 第一个点位置不变                         |

**被截取的 Path 片段会添加到 dst 中，而不是替换 dst 中到内容。**

**如果 startWithMoveTo 为 true, 则被截取出来到Path片段保持原状，如果 startWithMoveTo 为 false，则会将截取出来的 Path 片段的起始点移动到 dst 的最后一个点，以保证 dst 的连续性。**



### nextContour

我们知道 Path 可以由多条曲线构成，但不论是 getLength , getSegment 或者是其它方法，都只会在其中第一条线段上运行，而这个 `nextContour` 就是用于跳转到下一条曲线到方法，*如果跳转成功，则返回 true， 如果跳转失败，则返回 false。*



#### getPosTan

这个方法是用于得到路径上某一长度的位置以及该位置的正切值：

```
boolean getPosTan (float distance, float[] pos, float[] tan)
```

方法各个参数释义：

| 参数            | 作用                 | 备注                                                         |
| --------------- | -------------------- | ------------------------------------------------------------ |
| 返回值(boolean) | 判断获取是否成功     | true表示成功，数据会存入 pos 和 tan 中， false 表示失败，pos 和 tan 不会改变 |
| distance        | 距离 Path 起点的长度 | 取值范围: 0 <= distance <= getLength                         |
| pos             | 该点的坐标值         | 当前点在画布上的位置，有两个数值，分别为x，y坐标。           |
| tan             | 该点的正切值         | 当前点在曲线上的方向，使用 Math.atan2(tan[1], tan[0]) 获取到正切角的弧度值。 |



### getMatrix

这个方法是用于得到路径上某一长度的位置以及该位置的正切值的矩阵：

```
boolean getMatrix (float distance, Matrix matrix, int flags)
```

方法各个参数释义：

| 参数            | 作用                         | 备注                                                         |
| --------------- | ---------------------------- | ------------------------------------------------------------ |
| 返回值(boolean) | 判断获取是否成功             | true表示成功，数据会存入matrix中，false 失败，matrix内容不会改变 |
| distance        | 距离 Path 起点的长度         | 取值范围: 0 <= distance <= getLength                         |
| matrix          | 根据 falgs 封装好的matrix    | 会根据 flags 的设置而存入不同的内容                          |
| flags           | 规定哪些内容会存入到matrix中 | 可选择 POSITION_MATRIX_FLAG(位置)  ANGENT_MATRIX_FLAG(正切)  |



## Path & SVG

我们知道，用Path可以创建出各种个样的图形，但如果图形过于复杂时，用代码写就不现实了，不仅麻烦，而且容易出错，所以在绘制复杂的图形时我们一般是将 SVG 图像转换为 Path。

**开源库** ->[PathView](https://github.com/geftimov/android-pathview) 

**SVG 转 Path 的解析可以用这个库 ->[AndroidSVG](https://github.com/BigBadaboom/androidsvg) **