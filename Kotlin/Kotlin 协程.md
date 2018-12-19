# Kotlin 学习笔记：协程



Java并发 vs Kotlin 协程

线程有以下缺点

- Threads aren't cheap. Threads require context switches which are costly.
- Threads aren't infinite. The number of threads that can be launched is limited by the underlying operating system. In server-side applications, this could cause a major bottleneck.
- Threads aren't always available. Some platform, such as JavaScript do not even support threads
- Threads aren't easy. Debugging threads, avoiding race conditions are common problems we suffer in multi-threaded programming.

## 协程概念

本质上是轻量级线程。在Kotlin1.3版本中引入协程

相较于线程，有以下优点

- 最大的优势就是协程极高的执行效率。因为子程序切换不是线程切换，而是**由程序自身控制**，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。
- 第二大优势就是**不需要多线程的锁机制**，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。



 ## CoroutineScope

> Defines a scope for new coroutines. Every coroutine builder is an extension on [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) and inherits its [coroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html) to automatically propagate both context elements and cancellation.

​	定义了所创建的协程的范围。每个协程构建器都继承于它
 - launch方法：

> Launches new coroutine without blocking current thread and returns a reference to the coroutine as a [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html). The coroutine is cancelled when the resulting job is [cancelled](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html).

 - async方法

> Creates new coroutine and returns its future result as an implementation of [Deferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/index.html). The running coroutine is cancelled when the resulting deferred is [cancelled](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html).

```kotlin
Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function
```



delay
有点类似Thread.sleep()
只能在协程中使用

## runBlocking
协程构建器。创建新的协程，并阻塞当前线程，直到协程执行完毕

