# Matrix 原理



Matrix 是一个矩阵，最根本的作用就是坐标转换。

基本变换有4种: 平移(translate)、缩放(scale)、旋转(rotate) 和 错切(skew)。

下面我们看一下四种变换都是由哪些参数控制的。

![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/matrix01.png)



![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/matrix02.png)

### 缩放(Scale)



$$x ={k_1}{x_0}​$$

$y = {k_2}{y_0}$



用矩阵表示:

![img](http://latex.codecogs.com/png.latex?$$\left%20[%20\begin{matrix}%20x\\y\\1\end{1}%20\right%20]%20%20=%20\left%20[%20\begin{matrix}%20k_1%20%20&%20%20%200%20%20%20&%20%200%20%20\\%200%20%20%20&%20%20k_2%20%20&%20%200%20%20\\%200%20%20%20&%20%20%200%20%20%20&%20%201\end{1}%20\right%20]%20\left%20[%20\begin{matrix}%20x_0%20\\y_0%20\\1\end{1}%20\right%20]$$)

图例

![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/matrix03.jpg)

### 错切(Skew)

错切存在两种特殊错切，水平错切(平行X轴)和垂直错切(平行Y轴)。

#### 水平错切

$x={x_0}+ky_0$

$y=y_0$



用矩阵表示:

![img](http://latex.codecogs.com/png.latex?$$\left%20[%20\begin{matrix}%20x\\y\\1\end{1}%20\right%20]%20%20=%20\left%20[%20\begin{matrix}%20%20%201%20%20%20&%20%20k%20%20&%20%200%20\\%200%20%20%20&%20%201%20%20%20&%20%200%20\\%200%20%20%20&%20%200%20%20%20&%20%201\end{1}%20\right%20]%20\left%20[%20\begin{matrix}%20x_0\\y_0\\1\end{1}%20\right%20]$$)

图例:

![img](http://ww2.sinaimg.cn/large/005Xtdi2jw1f6cniifb0sj308c0dw3yz.jpg)

#### 垂直错切

![img](http://latex.codecogs.com/png.latex?$$%20x%20=%20x_0%20$$)

![img](http://latex.codecogs.com/png.latex?$$%20y%20=%20kx_0%20+%20y_0%20$$)

用矩阵表示:

![img](http://latex.codecogs.com/png.latex?$$\left%20[%20\begin{matrix}%20x\\y\\1\end{1}%20\right%20]%20%20=%20\left%20[%20\begin{matrix}%20%20%201%20%20%20&%20%200%20%20&%20%200%20\\%20k%20%20%20&%20%201%20%20%20&%20%200%20\\%200%20%20%20&%20%200%20%20%20&%20%201\end{1}%20\right%20]%20\left%20[%20\begin{matrix}%20x_0\\y_0\\1\end{1}%20\right%20]$$)

图例:

![img](http://ww4.sinaimg.cn/large/005Xtdi2jw1f6cnkwyksij308c0dwq3f.jpg)

#### 复合错切

> 水平错切和垂直错切的复合。

![img](http://latex.codecogs.com/png.latex?$$%20x%20=%20x_0%20+%20k_1%20y_0%20$$)

![img](http://latex.codecogs.com/png.latex?$$%20y%20=%20k_2%20x_0%20+%20y_0%20$$)

用矩阵表示:

![img](http://latex.codecogs.com/png.latex?$$\left%20[%20\begin{matrix}%20x\\y\\1\end{1}%20\right%20]%20%20=%20\left%20[%20\begin{matrix}%20%20%201%20%20%20&%20%20k_1%20&%20%200%20\\%20k_2%20&%20%201%20%20%20&%20%200%20\\%200%20%20%20&%20%200%20%20%20&%20%201\end{1}%20\right%20]%20\left%20[%20\begin{matrix}%20x_0\\y_0\\1\end{1}%20\right%20]$$)

图例:

![img](http://ww3.sinaimg.cn/large/005Xtdi2jw1f6cqdu6olfj308c0dwdgi.jpg)

### 旋转(Rotate)

假定一个点 A(x0, y0) ,距离原点距离为 r, 与水平轴夹角为 α 度, 绕原点旋转 θ 度, 旋转后为点 B(x, y) 如下:

![img](http://latex.codecogs.com/png.latex?$$%20x_0%20=%20r%20\cdot%20cos%20\alpha%20$$)

![img](http://latex.codecogs.com/png.latex?$$%20y_0%20=%20r%20\cdot%20sin%20\alpha%20$$)

![img](http://latex.codecogs.com/png.latex?$$x%20=%20r%20\cdot%20cos(%20\alpha%20+%20\theta)%20=%20r%20\cdot%20cos%20\alpha%20\cdot%20cos%20\theta%20-%20r%20\cdot%20sin%20\alpha%20\cdot%20sin%20\theta%20=%20x_0%20\cdot%20cos%20\theta%20-%20y_0%20\cdot%20sin%20\theta$$)

![img](http://latex.codecogs.com/png.latex?$$y%20=%20r%20\cdot%20sin(%20\alpha%20+%20\theta)%20=%20r%20\cdot%20sin%20\alpha%20\cdot%20cos%20\theta%20+%20r%20\cdot%20cos%20\alpha%20\cdot%20sin%20\theta%20=%20y_0%20\cdot%20cos%20\theta%20+%20x_0%20\cdot%20sin%20\theta$$)

用矩阵表示:

![img](http://latex.codecogs.com/png.latex?$$\left%20[%20\begin{matrix}%20x\\y\\1\end{1}%20\right%20]%20%20=%20\left%20[%20\begin{matrix}%20cos(\theta)%20&%20-sin(\theta)%20&%20%200%20\\sin(\theta)%20&%20cos(\theta)%20%20&%20%200%20\\0%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20&%20%20%20%20%20%20%200%20%20%20%20%20%20%20%20%20%20%20%20%20%20&%20%201\end{1}%20\right%20]%20%20.%20\left%20[%20\begin{matrix}%20x_0\\y_0\\1\end{1}%20\right%20]$$)

图例:

![img](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/matrix_rotation.jpg)

### 平移

![img](http://latex.codecogs.com/png.latex?$$%20x%20=%20x_0%20+%20\Delta%20x%20$$)

![img](http://latex.codecogs.com/png.latex?$$%20y%20=%20y_0%20+%20\Delta%20y%20$$)

用矩阵表示:

![img](http://latex.codecogs.com/png.latex?$$\left%20[%20\begin{matrix}%20x\\y\\1\end{1}%20\right%20]%20%20=%20\left%20[%20\begin{matrix}%201%20&%200%20&%20\Delta%20x%20\\0%20&%201%20&%20\Delta%20y%20\\0%20&%200%20&%201\end{1}%20\right%20]%20%20.%20\left%20[%20\begin{matrix}%20x_0\\y_0\\1\end{1}%20\right%20]$$)

图例:

![img](http://ww3.sinaimg.cn/large/005Xtdi2jw1f6dqiw20xoj308c0dw0su.jpg)

## Matrix 复合原理

其实Matrix的多种复合操作都是使用矩阵乘法实现的，从原理上理解很简单，但是，使用矩阵乘法也有其弱点，后面的操作可能会影响到前面到操作，所以在构造Matrix时顺序很重要。

我们常用的四大变换操作，每一种操作在Matrix均有三类,前乘(pre)，后乘(post)和设置(set)。

### 前乘(pre)

前乘相当于矩阵的右乘：

![img](http://latex.codecogs.com/png.latex?$$%20M%27%20=%20%20M%20\cdot%20S%20$$)

> 这表示一个矩阵与一个特殊矩阵前乘后构造出结果矩阵。

### 后乘(post)

后乘相当于矩阵的左乘：

![img](http://latex.codecogs.com/png.latex?$$%20M%27%20=%20%20S%20\cdot%20M%20$$)

> 这表示一个矩阵与一个特殊矩阵后乘后构造出结果矩阵。

### 设置(set)

设置使用的不是矩阵乘法，而是直接覆盖掉原来的数值，所以，**使用设置可能会导致之前的操作失效**。