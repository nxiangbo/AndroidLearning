# 自定义View之Path

> The Path class encapsulates compound (multiple contour) geometric paths
> consisting of straight line segments, quadratic curves, and cubic curves.
> It can be drawn with canvas.drawPath(path, paint), either filled or stroked
> (based on the paint's Style), or it can be used for clipping or to draw
> text on a path.

Path类封装了由直线段，二次曲线和三次曲线组成的复合（多个轮廓）几何路径。 它可以使用canvas.drawPath（path，paint）绘制，填充或描边（基于绘制的样式），或者它可以用于剪切或在路径上绘制文本。

在自定义View的过程中，Path是非常有用的。通过它可以绘制任何直线，贝塞尔曲线，以及添加圆形或者矩形等。

