# Android 动态代理和hook机制

本文主要参考[Android插件化原理解析——Hook机制之动态代理](http://weishu.me/2016/01/28/understand-plugin-framework-proxy-hook/)

代理分为静态代理和动态代理。

静态代理是为每个被代理对象创建代理类。静态代理的代理类与被代理类实现相同的接口或者继承同一父类。

而动态代理是可以在代码运行时自动创建相应的代理类。

Java中提供了Proxy和InvocationHandler两个API。Proxy可以通过调用newProxyInstance方法获取一个代理类实例。InvocationHandler是一个接口。每个实现InvocationHandler接口的类会持有对应的被代理类实例。



可以使用代理进行hook。在插件化中，宿主加载插件时需要考虑插件生命周期的问题。因此，为了激活插件Activity的生命周期，hook Instrumentation。具体步骤是，首先通过反射得到ActivityThread类，同时，根据currentActivityThread方法获得ActivityThread对象。其次，通过ActivityThread对象拿到mInstrumentation的成员变量。然后，自定义一个CustomInstrumentation类（继承于Instrumentation）并将这个对象替换掉原来的Instrumentation对象。最后，调用Instrumentation的任何方法都可以通过CustomInstrumentation来进行拦截或修改。

```java
Class<?> activityThreadClass = Class.forName("android.app.ActivityThread");
		Method currentActivityThreadMethod = activityThreadClass.getDeclaredMethod("currentActivityThread");
		currentActivityThreadMethod.setAccessible(true);
		Object currentActivityThread = currentActivityThreadMethod.invoke(null);

		Field mInstrumentationField = activityThreadClass.getDeclaredField("mInstrumentation");
		mInstrumentationField.setAccessible(true);
		Instrumentation mInstrumentation = (Instrumentation) mInstrumentationField.get(currentActivityThread);

		Instrumentation newInstrumentation = new NewInstrumentation(mInstrumentation);
		mInstrumentationField.set(currentActivityThread, newInstrumentation);
```





