Activity从创建到销毁系统帮我们做了哪些事



## 实例化Activity

在ActivityThread中的方法通过反射创建Activity对象，同时，创建ContextImp，PhoneWindow，并attach到Activity。

## onCreate

回调onCreate方法。在onCreate方法中用setContentView加载布局。setContentView 主要将