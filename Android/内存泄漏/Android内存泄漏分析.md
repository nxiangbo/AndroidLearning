# Android内存泄漏

## 内存泄漏

在java中，虽然jvm提供了垃圾回收机制，但仍需注意对象的使用不当，造成内存一直被持有，而不可被回收，导致内存泄漏的问题。
何时会内存泄漏呢？
jvm垃圾回收机制使用可达性分析的算法。对象不可用，但仍被其他对象引用。




## Android常见内存泄漏场景


### 非静态内部类引用外部类导致内存泄漏

以Handler为例，在Activity中创建一个Handler，并创建一个延时任务。当在任务处理完成前，Activity被销毁时，gc过程发现Handler引用着Activity对象，不会回收。因此该Activity不能被正确回收导致内存泄漏。

#### 内存泄漏原因

Handler内存泄漏的引用关系

- 非静态内部类引用着外部类（Handler -> Activity）
- 入消息队列时，消息msg.target=Handler，Message持有Handler（Message -> Handler）


```java
MessageQueue -> Message -> Handler -> Activity
```

何时Message断开Handler引用呢？
当从消息队列中取出Message后，执行消息任务时，会调msg.recycleUnchecked将target置为null。

#### 解决方案

弱引用 + 静态内部类


### context内存泄漏
单例对象持有Activity的context。

#### 内存泄漏原因

由于单例的生命周期与应用程序进程一致，而一般Activity生命周期要短。当Activity被destroy后，单例仍持有Activity的context，不会被gc回收掉，导致内存泄漏。

#### 解决方案
将Activity的context替换为application的context


### I/O流或数据库等未关闭




## 分析内存泄漏工具

### leakcannary

LeakCanary可以自动检测Activity/Fragment对象内存泄漏，并生成堆转储文件。

如何确认Activity/Fragment内存泄漏？

1. 利用LifecycleCallback监听Activity的onDestroy
2. 将Activity包装为WeakReference
3. 主动调用gc，检测引用队列中是否有该Activity，如果不在引用队列中，则内存泄漏。

*ReferenceQueue：当对象被回收后，其相应的包装类（Reference对象）会被放入到引用队列中*

### Android Profile


### MAT

