# png图片工作原理

png的全称是 [Portable Network Graphics](https://en.wikipedia.org/wiki/Portable_Network_Graphics)，它是一种图片格式，应用非常广泛。png提供了高质量的图片，因此，所占空间比较大。

## 图片压缩

png的压缩方式是无损压缩。这意味着被压缩的文件可以重建源文件。

压缩分两步：预测和压缩

### 预测（过滤）

差分编码（delta encoding）又称**增量编码**，是以[序列](https://zh.wikipedia.org/wiki/%E5%BA%8F%E5%88%97)式数据之间的差异存储或发送数据的方式（相对于存储发送完整文件的方式）。

[2,3,4,5,6,7,8]通过差分编码可以变成[2,1,1,1,1,1,1], 其中

[2, 3–2=1, 4–3=1, 5–4=1, 6–5=1, 7–6=1, 8–7=1]

如果数据是线性相关的，那么会将数据转换为大量重复的数据值。可以更大程度的进行压缩。

举例来讲。

![img](https://cdn-images-1.medium.com/max/880/0*9ZmTLC3rBj95db63.)





![img](E:\AndroidLearning\图片\images\1_9DREStZ3sX45FZFt_SwDfg.png)

59-（55+63）/2 = 0

55-(61+59)/2 = -5

90-(55+66)/2 = 30

109-(70+90)/2 = 29



现在，值得注意的是每条线可能会有所不同，这就是为什么PNG允许每行选择5种不同模式中的一种：

- 不过滤
- X-A
- X-B
- X-(A+B)/2
- [Paeth](https://www.w3.org/TR/PNG-Filters.html) predictor (linear function of A,B,C)

![img](E:\AndroidLearning\图片\images\1_x-yXQnUD4Vjjq10ALPyirQ.png)



### 压缩（DEFLATE算法）

在扫描线上发生过滤后，就会应用DEFLATE算法进行无损压缩。**DEFLATE**是同时使用了LZ77算法与哈夫曼编码（Huffman Coding）的一个无损数据压缩算法。 这几乎与PKWARE，PKZIP，GZip等压缩器相同。实现是开箱即用的标准，但在图像数据方面有一些有趣的警告。

- DEFLATE限制匹配3到258个符号之间的长度; 最大可设想的压缩比约为1032：1。
- 如果匹配小于3个符号，则会产生一些开销来表示符号

看下面这个图像。270 x 90 大小是20KB。而270 x 92 大小是41KB

![img](E:\AndroidLearning\图片\images\0_EYssBxmJP27Gya81_.jpg)



Deep blue = very compressed, yellow/red = not very compressed





![img](E:\AndroidLearning\图片\images\0_XQUFZC5ajMekUkkl_.jpg)



这里发生的是，在较小的图像中，扫描线中有更多匹配，因此压缩效果更好。 但是，稍微调整大小会改变可能发生的匹配类型; 一些潜在的匹配候选者现在在我们的LZ窗口之外，因此不匹配，导致更大的文件。



## PNG文件格式分析

现在，值得注意的是PNG不仅仅是过滤和压缩阶段。 它是一种非常可扩展的容器格式，可以支持各种图像类型和添加的数据。PNG文件格式里面包含不同的区块（chunks），各个区块带有不同类型的数据。

 Png格式是由四个关键数据块和一些辅助数据块组成，四个关键数据块是必需的。

关键数据块包括：

- 文件头数据块（IHDR）包含有图片的宽度、高度、位的深度和颜色类型
- 调色板数据块（PLTE）包含有与索引彩色图像(indexed-color image)相关的彩色变换数据，它仅与索引彩色图像有关。PLTE数据块是定义图像的调色板信息，PLTE可以包含1~256个调色板信息，每一个调色板信息由3个字节组成
- 图象数据块（IDAT）包含了实际的图像信息（数据块可以有多个）
- 图象结束数据块（IEND）：它用来标记PNG文件或者数据流已经结束，并且必须要放在文件的尾部

其他数据块

- 默认的背景色
- 色度如何显示白点上的坐标
- 图像γ数据块
- 文本数据，语言或元数据信息
- 颜色空间信息
- 立体图像数据
- 图像最后修改时间数据块
- 图像透明数据块

上面的这些区块是你特别需要注意的地方，因为很多垃圾信息是在图片编辑软件编辑后添加进去的。举个例子，你在Photoshop保存了一张PNG格式图片，这行图片里就会有一个区块记录着"这张图片是由photshop创建的"。这样的信息对于一张显示给人看的图片并没什么鸟用，但它还是会包含在图片里。因此，删除没有的区块是减少文件大小的关键。下面是一张16X16像素的图片，左边那张是Photoshop保存的普通图片，右边那张是用了photoshop的"导出web格式"选项保存的图片，这个选项会去掉了所有的附加信息。

![img](E:\AndroidLearning\图片\images\298426-635f087e317eea89.webp)



## 像素格式

PNG也支持很多种类型的像素格式，你可以选择一种最佳的：

- Indexed(索引色) = 1个频道，可以为1，2，4，8 bpc()
- Grayscale(灰度) = 1个频道，可以为1，2，4，8，16 bpc
- Gray+Alpha(带透明通道的灰度) = 2个频道，可以为8或16 bpc
- Truecolor(RGB 真彩色) = 3个频道，可以为8或16 bpc
- RGBA(RGBA 带透明通道的真彩色) = 4个频道，可以为8或16 bpc





## 减小PNG图片大小

如果你理解PNG格式，有一些点是明显可以优化的：

- 去掉不需要的区块(chunks)
- 减少特殊颜色
- 优化每一行的过滤算法
- 优化DEFLATE压缩率

有很多png优化工具可以用。如tinyPNG等



### 减少颜色

在PNG压缩过程中的**过滤**(filtering)阶段，压缩的强度是取决于相邻像素色的差异程序。例如：减少特殊颜色的数量会减少相邻像素色的差异程序，继而减小过滤出来后的值的大小。因此，到了**DEFLATE压缩**阶段就会得到更多重复的值，当然就能压缩得更小。

### 选择正确的像素格式

例如，对于没有alpha通道的图片，不应该使用RGBA格式的png图片。应该将其转换为jpg格式或者使用24bpp的真彩色(RGB)格式。

### 索引图片

颜色的减少应该始终从尝试优化你的颜色数开始，那么就可以把图片定义为**索引格式**(INDEXED FORMAT)了。索引颜色的原理是，只选择使用最佳的256种颜色，然后把你的图片的所有像素都替换成这256种颜色。想想看从原来的1600万种颜色(24bpp)减少到256种，这会减少多少空间呀。

![img](E:\AndroidLearning\图片\images\298426-339be9c3d234eba5.webp)

上面这张Google涂鸦是从Photoshop的"save for web"选项导出来的，它的图片格式被设置为PNG8(8bpp)，它的颜色都来自下面的调色板：



![img](E:\AndroidLearning\图片\images\298426-23de6345c0baf497.webp)

把真彩图转成索引图，你就得把每一个特殊的颜色都替换成调色板里的颜色，这样做能把占32位的一个像素减少到8位，这一步就减少了很多空间。

这个模式还会在过滤和压缩阶段带来进一步的节省：

1. 由于特殊颜色已经减少，这意味着相邻的颜色有更高概率是同样的颜色
2. 又由于相邻同样像素颜色数量增加，在过滤阶段会产生更多重复的值，在使用LZ77的DEFLAFT算法的压缩效果会更好





## aapt 优化png图片

AAPT工具可能会在构建期间自动对res/drawable/文件夹下的图片资源做无损压缩。例如，一张真彩色(RGB)的PNG图片，如果它实际需要的颜色不大于256种，那么就有可能会被转换成一张8位的调色板图片(PNG8)。转换之后的图片质量未减却只占用更小的内存。所以需要注意一点，在这个目录下的二进制图像在构建期间可能会被改变。

### aapt工具如何优化图片

Android系统其中的一个好处是它的大部分工具都是开源的，我们可以直接看AAPT的[源码](https://link.jianshu.com?t=https://android.googlesource.com/platform/frameworks/base/+/marshmallow-dev/tools/aapt2/)，看看它到底做了什么。在[png.cpp](https://link.jianshu.com?t=https://android.googlesource.com/platform/frameworks/base/+/marshmallow-dev/tools/aapt2/Png.cpp)文件里定位到**analyze_image**方法，我们可以看到它会做三个优化检查：

1. 每个像素都是 R == G == B (grayscale 灰度)
2. 每个像素都是 A == 255 (opaque 全透明)
3. 是否有超过256种不同的RGBA颜色

所以APPT会加载PNG图片，判断它是否可以被转成灰度格式的图片，判断它是否是全透明的，或判断它是否可以被转成一张索引图。这就是AAPT干的事。

这些检测是很有意义的，因为实际上APK里的大部分PNG文件通常都是灰度图，全透明，或有限颜色的图片。在有能力知道这些图片的格式，并把它们转成更小的PNG文件，何乐而不为呢。



### 注意

[递归数据压缩是不可能的](https://link.jianshu.com/?t=http://goodmath.scientopia.org/2010/08/03/revenge-of-the-return-of-the-compression-idiot/)。如果你对数据压缩过一次，然后想再压缩一次，最好的情况是数据大小不变，但通常情况会得到一个更大的数据。所以，如果你在把PNG图片扔给AAPT前已经做了一次优化，[那么最终可能会变得更大](https://link.jianshu.com?t=https://code.google.com/p/android/issues/detail?id=65335)，大概[大10%](https://link.jianshu.com?t=http://stackoverflow.com/questions/12929884/disable-android-resource-image-png-optimization)。这似乎不太正常：你都已经花了那么多时间压缩了你的PNG图片了，它们后来怎么还能变大呢:\

解决这个问题的办法是在Gradle文件里用cruncherEnabeled选项去掉对PNG文件的处理：

```
aaptOptions {
    cruncherEnabled = false
}
```






























### 什么是Alpha Channels

也被称为mask channel，它是一种将透明度与图片相关联的方式。

Also known as a *mask channel*, an alpha channel is simply a way to associate variable transparency with an image. Whereas GIF supports simple binary transparency--any given pixel can be either fully transparent or fully opaque--PNG allows up to 254 levels of partial transparency in between for "normal" images (or 65,534 levels for the special "deeply insane" formats, but here we're concentrating on image depths that are useful on the Web).

All three PNG image types--truecolor, grayscale and palette--can have alpha information, but it's most commonly used with truecolor images. Instead of storing three bytes for every pixel (red, green and blue), now four are stored: red, green, blue and alpha, or RGBA. The variable transparency allows you to create "special effects" that will look good on any background, whether light, dark or patterned. For example, a photo-vignette effect can be created for a portrait by making a central oval region fully opaque (i.e., for the face and shoulders), the outer regions fully transparent, and a transition region that varies smoothly between the two extremes. When viewed with a Web browser such as Arena, the portrait would fade smoothly to white when viewed against a white background, or smoothly to black if against a black background. Drop-shadows are another ideal application for alpha transparency; in the images below, the same toucan image is displayed against a colorful background and against another copy of itself:



![[RGBA toucan viewed with rpng2 -bgpat 14]](http://www.libpng.org/pub/png/img_png/rpng2-bg14-toucan.png)   ![[RGBA toucan overlaid on itself against a pink background]](http://www.libpng.org/pub/png/img_png/toucans.jpg)
*Stefan Schneider's shadow-casting toucan displayed against different backgrounds*

This transparency feature is far more important for the small web graphics that are typically used on web pages, such as colored (circular) bullets and fancy text. Alpha blending allows one to use *anti-aliasing*--creating the illusion of smooth curves on a grid of rectangular pixels by smoothly varying the pixels' colors--to make rounded and curved images that look good against *any* background, not just against a white background (for example). Thus the same image can be reused in many places without the "ghosting" effect that occurs with GIFs.

Of course, effective replacements for GIF buttons and icons must be comparable in size as well, and that mostly rules out truecolor RGBA images. But PNG supports alpha information with palette images as well; it's just slightly harder to implement in a smart way. A PNG alpha-palette image is just that: an image whose palette also has alpha information associated with it, not a palette image with a full alpha mask. In other words, each pixel corresponds to an entry in the palette with red, green, blue *and* alpha components. So if you want to have bright red pixels with four different levels of transparency, you must use four separate palette entries to accommodate them. (All four entries will have identical RGB components, but the alpha values will differ.) If you want *all* of your colors to have four levels of transparency, you've effectively reduced your total number of available colors from 256 to 64. In general, though, only some of the colors need more than one level of transparency, and recognizing which ones is where things get tricky for the programmer. (If you don't want to trust your local programmer, have a look at [**pngquant**](http://www.libpng.org/pub/png/apps/pngquant.html), which converts 32-bit RGBA PNGs into 8-bit RGBA-palette images. If you *are*a programmer, also have a look at it; full source code is included.)

