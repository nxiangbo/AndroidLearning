# webp 图片格式原理

[探究WebP一些事儿](https://aotu.io/notes/2016/06/23/explore-something-of-webp/index.html)

[WebP原理和Android支持现状介绍](https://zhuanlan.zhihu.com/p/23648251)

- webp 的图片比jpeg和png的图片更小，加载效率更高。

- webp支持有损压缩，无损压缩以及有损压缩（支持透明）

所用到的压缩技术

[Compression Techniques](https://developers.google.com/speed/webp/docs/compression)



## 简介

在2013年，Google(和一些开源贡献者)创建了一种新的图片编解码算法，叫做WebP，它旨在同样的的图片质量下比JPG压缩得更小。一张同样大小和复杂度的图片，WebP可以比JPG小24%-35%。

WebP文件格式来源于[VP8视频编解码](https://link.jianshu.com?t=http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/pubs/archive/37073.pdf)(你可能更知道[WebM](https://link.jianshu.com?t=http://www.webmproject.org/))。VP8编解码器的其中一个强大特性是[帧内预测压缩](https://link.jianshu.com?t=http://blog.webmproject.org/2010/07/inside-webm-technology-vp8-intra-and.html)，或者说，视频的每一帧都被压缩，后续帧与帧之间的差异也会被压缩。

这就是WebP的由来：WebM文件里单个被压缩的帧。

或者，更精确的说WebP的*核心来自WebM*。自从2011年发布以来，WebP作为一个独特文件类型它也做了很多改变和升级。例如像透明度，无损模式，和一些诡异扭曲，以及对动画的支持。

WebP的优势在于它具有更优的图像数据压缩算法，在拥有肉眼无法识别差异的图像质量前提下，带来更小的图片体积，同时具备了无损和有损的压缩模式、Alpha 透明以及动画的特性，在 JPEG 和 PNG 上的转化效果都非常优秀、稳定和统一。





## 原理



WebP的压缩主要分为**有损压缩**、**无损压缩**以及**有损带透明通道压缩**。

## 有损压缩

### 宏块

编码器的第一个阶段是将图片分割成不同"宏块"。典型的宏块包括一个16x16的亮度像素块，和两个8x8的色度像素块。这个阶段非常像JPEG格式里转换颜色空间，对色度通道降低采样，以及细分图片。

![img](E:\AndroidLearning\图片\images\298426-07c6dd948b262797.webp)

### 预测

宏块里每个4x4的子块都有一个预测模型。(又名[过滤](https://link.jianshu.com/?t=https://en.wikipedia.org/wiki/Delta_encoding))。在PNG里过滤用得非常多，它对每一行都做同样的事，而WebP过滤的是每一块。它是这样处理的，在一个块周围定义两组像素:有一行在它上面为A，在它左边那一列为L。

![img](E:\AndroidLearning\图片\images\298426-57125684115c7c24.webp)

Horiz prediction(水平预测)——将块的每列使用左列（L）数据的副本进行填充。

Vertical Prediction(垂直预测)——将块的每行使用上列（A）数据的副本进行填充。

DC Prediction(DC 预测)——将块使用 A 上列的像素与 L 左列的像素的平均值进行填充。

True Motion (TrueMotion 预测)



### 剩下步骤与jpg压缩类似



WebP编码的最后阶段看起来非常像我们的[老朋友JPG](https://link.jianshu.com?t=https://medium.freecodecamp.com/how-jpg-works-a4dbd2316f35#.st8n7dcuh):

- 对块里剩余的值执行DCT过滤
- DCT矩阵后量化
- 转成量化矩阵后重新排序，然后送到一个静态压缩器里。



![img](https:////upload-images.jianshu.io/upload_images/298426-5298a2ded42c27a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/624/format/webp)



这有主要有两点不同：

1. 在DCT阶段输入的数据不是原始的数据块本身，而是预测后的数据
2. WebP用得静态压缩器是[算术压缩器](https://link.jianshu.com?t=https://www.youtube.com/watch?v=FdMoL3PzmSA&index=7&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H)，它和JPG用的[霍夫曼编码器](https://link.jianshu.com?t=https://www.youtube.com/watch?v=6rnF2Mo80x0&feature=youtu.be&t=16m3s)类似。



## 应用

在Android开发中，有损的WebP图片需要Android minSdk 在4.0（API level 14）及以上，而无损和带透明度的WebP图片需要Android 4.3（API level 18）及以上。



### 如何将PNG转换为WebP



Android Studio也提供了将图片转换为webP格式的工具。

此外，谷歌提供了一个命令行工具webp，用于将图片转换为webp格式。

[工具文档](https://developers.google.com/speed/webp/docs/using)



