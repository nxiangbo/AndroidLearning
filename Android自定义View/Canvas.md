# Canvas



> The Canvas class holds the "draw" calls. To draw something, you need
4 basic components: A Bitmap to hold the pixels, a Canvas to host
the draw calls (writing into the bitmap), a drawing primitive (e.g. Rect,
Path, text, Bitmap), and a paint (to describe the colors and styles for the
drawing).



Canvas 主要提供了在画布上绘制的功能。它有一系列的drawXXX()方法。

当然，Canvas的功能不止于此，它还有裁切指定范围的功能，对应的是clipXXX方法。

它还具有几何变换的功能。平移，旋转，缩放，也可以使用Matrix自定义变换规则，setMatrix(Matrix matrix) concat(Matrix matrix)。



save()和restore()方法必须成对使用。

save(): 保存当前Canvas的状态，包括matrix和clip。后续调用translate，scale，rotate，skew，concat或者clipRect和clipPath 正常使用

restore(): 删除所有的修改，防止save()方法代码之后对Canvas执行的操作，继续对后续的绘制会产生影响，通过该方法可以避免连带的影响 

saveLayer和saveLayerAlpha方法。

在Canvas的drawXXX方法中，有一个drawPath的方法。可以通过Path绘制各种曲线，包括贝塞尔曲线。

## Path


> The Path class encapsulates compound (multiple contour) geometric paths
consisting of straight line segments, quadratic curves, and cubic curves.
It can be drawn with canvas.drawPath(path, paint), either filled or stroked
(based on the paint's Style), or it can be used for clipping or to draw
text on a path.







