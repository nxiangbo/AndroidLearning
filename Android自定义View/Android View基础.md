#  Android View 基础

> 他山之石，可以攻玉



## View 坐标系

View的坐标是相对于父控件而言的。

```java
getTop();       //获取子View左上角距父View顶部的距离
getLeft();      //获取子View左上角距父View左侧的距离
getBottom();    //获取子View右下角距父View顶部的距离
getRight();     //获取子View右下角距父View左侧的距离
```



![View 坐标系](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/view坐标.jpg)

## MotionEvent中 get 和 getRaw 的区别

```java
event.getX();       //触摸点相对于其所在组件坐标系的坐标
event.getY();

event.getRawX();    //触摸点相对于屏幕默认坐标系的坐标
event.getRawY();
```



![](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/view02.jpg)

## View 绘制流程

自定义View绘制流程示意图

![](/Users/nxiangbo/Documents/AndroidLearning/Android自定义View/images/view03.jpg)



### 构造方法

View 有四个构造方法。

```java
public void SloopView(Context context) {} // 一般直接new SlooView（context）时使用
public void SloopView(Context context, AttributeSet attrs) {} // 一般在xml布局中使用
public void SloopView(Context context, AttributeSet attrs, int defStyleAttr) {}
public void SloopView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}
```



### onMeasure

**View的大小不仅由自身所决定，同时也会受到父控件的影响，为了我们的控件能更好的适应各种情况，一般会自己进行测量。**



```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthsize  MeasureSpec.getSize(widthMeasureSpec);      //取出宽度的确切数值
    int widthmode  MeasureSpec.getMode(widthMeasureSpec);      //取出宽度的测量模式
    
    int heightsize  MeasureSpec.getSize(heightMeasureSpec);    //取出高度的确切数值
    int heightmode  MeasureSpec.getMode(heightMeasureSpec);    //取出高度的测量模式
}
```

widthMeasureSpec和heightMeasureSpec中既包含view的宽高信息（低30位），也包含了测量view的模式（高2位）。


| 模式        | 二进制数值 | 描述                                                         |
| ----------- | ---------- | ------------------------------------------------------------ |
| UNSPECIFIED | 00         | 默认值，父控件没有给子view任何限制，子View可以设置为任意大小。 |
| EXACTLY     | 01         | 表示父控件已经确切的指定了子View的大小。                     |
| AT_MOST     | 10         | 表示子View具体大小没有尺寸限制，但是存在上限，上限一般为父View大小。 |





### onSizeChanged

在view大小改变时调用。一般在这个函数中确定view的大小。

```java
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
}
```



### onLayout

**它用于确定子View的位置，在自定义ViewGroup中会用到，他调用的是子View的layout函数。**

在自定义ViewGroup中，onLayout一般是循环取出子View，然后经过计算得出各个子View位置的坐标值，然后用以下函数设置子View位置。

```java
child.layout(l, t, r, b);
```



### onDraw

onDraw是实际绘制的部分，使用的是Canvas绘图。

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
}
```



draw的流程：
1. 绘制背景
2. If necessary, save the canvas' layers to prepare for fading
3. 绘制View内容
4. 绘制子View
5. If necessary, draw the fading edges and restore layers
6. Draw decorations (scrollbars for instance)



 ## invalidate（）
invalidate系列方法请求重绘View树（也就是draw方法），如果View大小没有发生变化就不会调用layout过程，并且只绘制那些“需要重绘的”View，也就是哪个View(View只绘制该View，ViewGroup绘制整个ViewGroup)请求invalidate系列方法，就绘制该View。

      

## requestLayout（）
requestLayout()方法会调用measure过程和layout过程，不会调用draw过程，也不会重新绘制任何View包括该调用者本身。
View#requestLayout()

```java
public void requestLayout() {
        ......
        if (mParent != null && !mParent.isLayoutRequested()) {
            //由此向ViewParent请求布局
            //从这个View开始向上一直requestLayout，最终到达ViewRootImpl的requestLayout
            mParent.requestLayout();
        }
        ......
    }
```
当我们触发View的requestLayout时其实质就是层层向上传递，直到ViewRootImpl为止，然后触发ViewRootImpl的requestLayout方法，如下就是ViewRootImpl的requestLayout方法：

```java
@Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            //View调运requestLayout最终层层上传到ViewRootImpl后最终触发了该方法
            scheduleTraversals();
        }
    }
```