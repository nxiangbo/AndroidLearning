# jpg 工作原理



JPG的压缩方案被分成几个步骤，下图很好的把它们展示出来，我们接下来会对每个步骤作详细介绍。



![img](images\jpg工作原理图.png)


### 色彩空间转换
一个色彩空间是一个特定的颜色组织，它的颜色模型代表了这些颜色表示的数学公式。(例如三颜色RGB，或者CMYK)。

这个过程的强大之处是，你可以把一个颜色模型转换成另外一个，这意味着你可以将一个给定颜色的数学表达式转成一组完全不同的值。

将JPG从RGB颜色模型转换成[Y,Cb,Cr](https://link.jianshu.com/?t=https://en.wikipedia.org/wiki/YCbCr)颜色模型，包括亮度(Y)、色度蓝(CB)和色度红(CR)。

![img](images/webp)

### 缩减取样

CbCr颜色空间其中一个有趣的细节是，Cb/Cr通道有比较少的颗粒度细节，也就是说它们包含的信息没有Y通道的多。所以JPG的算法就把Cb和Cr通道的大小缩减到原来的1/4，这种做法就叫做*缩减取样*。

有一件很重要的事需要说明，*缩减取样*是一个有损压缩过程(压缩过后你无法恢复回原来的颜色，只能得到一个近视的值)，但对人类视觉皮层的可视化组件的整体影响是最小的。由于我们没有对亮度(Y)做处理，只对CbCr通道做缩减，所以对人的视觉系统的影响较低。

![img](E:\AndroidLearning\图片\images\webp2)



### 图像分成8x8像素块

从现在开始，在JPG上做的所有操作都是基于8x8像素块。这样做是因为我们通常认为8x8像素块里没有太多的差异，即使是很复杂的图片，局部区域的像素也往往相似，这种相似性有助于之后我们对它压缩。



### 离散余弦变换

DTC([离散余弦转换](https://link.jianshu.com/?t=https://zh.wikipedia.org/wiki/%E7%A6%BB%E6%95%A3%E4%BD%99%E5%BC%A6%E5%8F%98%E6%8D%A2))的关键是，它假设任何一个数字信号都可以被一个余弦组合函数重建。



核心思想是，在不同频率上，8x8块都可以由一组权重余弦变化的和代表。这里的重点是算出要输入的余弦值，和它们的权重。

主要参考wiki https://en.wikipedia.org/wiki/JPEG





![img](E:\AndroidLearning\图片\images\0_o3zmZjKGMOUgmJac_.png)



举例

每个色彩通道的8x8像素块都会用上面这个公式和基础函数生成新的一个8x8矩阵，表示在这个过程中使用的权重。下面是一张表示这个过程的图片:

![img](E:\AndroidLearning\图片\images\298426-11df025db578e2b2.webp)



这个矩阵G，用来代表重建图片的基础权重(在动画右侧上方的小十进制数)。对于每个基础余弦值，我们都将它乘与矩阵里的权重，然后全部相加，得到我们最终的图片。







### 量化

我们不想压缩浮点数据，它会膨胀我们的信息流，并且不高效。因此，我们要找一种方法来将权重矩阵转换到0-255这范围里。我们可以直接在矩阵里将最小/最大值(分别是-415.38和77.13)除以这个范围来得到一个在0-1的值，再将这个值乘以255得到最后的值。

例如:[34.12- -415.38] / [77.13 — -415.38] *255= 232



这样做可以，但代价是精度会显著下降。这种缩放将产生一个分布不均匀的值，其结果是会导致图片质量的下降。

JPG用了另外一种方法。不同于使用矩阵的范围来作为缩放值，JPG用了一个可量化的预矩阵。这种可量化矩阵不需要成为数据流的一部分，它们可成为编码的一部分。

[这个例子](https://link.jianshu.com?t=https://en.wikipedia.org/wiki/JPEG#Quantization)展示了对于每一张图片使用的量化因子矩阵:



![img](https:////upload-images.jianshu.io/upload_images/298426-7e526e0cadbd0a8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/335/format/webp)



现在我们使用Q和G矩阵，来计算我们的量化DCT系数矩阵：



![img](https:////upload-images.jianshu.io/upload_images/298426-9c3e6881ded3f49c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/501/format/webp)

12.png

例如，用G矩阵的[0,0]=-415.37和Q[0,0]=16这两个数字:



![img](https:////upload-images.jianshu.io/upload_images/298426-61f6c004bf513e1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/376/format/webp)

13.png

得到最后的矩阵是：



![img](https:////upload-images.jianshu.io/upload_images/298426-74a34f4783584d65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/390/format/webp)



你看现在这个矩阵多变成多简单，它现在包含了大量的小整数或者0，这会有助于压缩。

简单点说，我们把这个过程分别应用在Y,CbCr通道，这样我们需要两个不同的矩阵，一个是Y通道，另一个是C通道:



![img](https:////upload-images.jianshu.io/upload_images/298426-da71693ee2e443e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/335/format/webp)

![img](E:\AndroidLearning\图片\images\0_PbwrOzC1Dd0gkwdV_.png)



量化压缩有两种重要的方式：

1. 限制可用范围的权重，减少数字的位数和替换它们。
2. 将大部分的权重变成同一个数字或者0，提高第三步压缩，熵编码。

这样的量化方式主要来源于JPEG。因为图像的右下方往往有最大的量化因子，JPEG倾向于将这些图像组合。量化因子矩阵可以通过改变JPEG的'质量值'直接控制。(稍后介绍)

## 压缩

现在我们可以回到整数世界了，将目光转移到对颜色块做有损压缩阶段。当看我们的转化数据，你应该会注意到一些有趣的现象：





![img](https:////upload-images.jianshu.io/upload_images/298426-3078064febdd60e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/390/format/webp)

从左上角到右下角，0出现的数量急速上升。但主行和主列的顺序并不理想，因为这些0都交织在一起，而不是放在一起。

相反，我们可以从左上角开始以Z字形来回穿梭直到右下角的方式遍历整个矩阵。







![img](E:\AndroidLearning\图片\images\0_gK0Zt5sPdspjTTrO_.png)



我们的亮度矩阵的顺序变成：

*−26,−3,0,−3,−2,−6,2,−4,1,−3,1,1,5,1,2,−1,1,−1,2,0,0,0,0,0,-1,-1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0*

一旦数据变成这个格式，下一步就简单了：按顺序执行RLE，然后对结果进行统计编码([霍夫曼](https://link.jianshu.com?t=https://www.youtube.com/watch?v=6rnF2Mo80x0&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H&index=2)/[算术](https://link.jianshu.com?t=https://www.youtube.com/watch?v=FdMoL3PzmSA&index=7&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H)/ANS)。

Boom.你的数据块变成JPG编码了。

## 理解质量参数

现在你已经理解JPG文件是怎样创建的，是该重新审视图片质量参数这个概念了，你通常会在PhotoShop导出JPG图片时会看到。(或在其他地方)

这个参数我们称之为*q*，是一个从1到100的整数。你应该把q当作一个测量图片质量的值:q越大表示图片的质量越高，当然文件就越大。

**这个质量值用在量化阶段，用来缩放合适的量化因子。**因此对于每个基准权重，量化阶段现在类似于** round(Gi,k / alpha*Qi,k)。**

其中alpha符号用来当作质量参数。



![img](https:////upload-images.jianshu.io/upload_images/298426-bccc2c5883e4b5cc.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/218/format/webp)



当alpha或Q[x,y]越大(当alpha的值越大那么q参数的值就越小)，更多信息是丢失，文件就越小。

所以，如果你想要靠更多的视觉假象来得到一张更小的图片，那么你可以在到处图片阶段设置一个更小的质量值。



![img](https:////upload-images.jianshu.io/upload_images/298426-fd9dc507c74afaf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/497/format/webp)



注意上图，那张质量为1的图片里，我们可以清晰的看到在成块阶段和量化阶段的痕迹。

更重要的是质量参数大小取决于具体的图片。由于每张图片都是唯一的，它们都有不同的展示效果，那么Q值也是唯一的。





## 优化jpg图片

设置合理的quality值

模糊色度

转换为webp