# Kotlin 学习笔记：协程

> 保持一个上升的态势比什么都重要



## 协程概念

### 什么是协程

本质上是轻量级线程。

目标

可控制：协程能做到可被控制的发起⼦任务
轻量级：协程非常轻量级，占用资源比线程少
语法糖：使多任务或多线程切换不再使用回调语法

### 协程vs线程

线程

1. 线程切换开销较大
2. 可启动线程数量有限，如果开启很多线程，容易OOM
3. 线程调试比较困难

协程

1. 相较于线程，协程之间切换开销非常非常小

2. 轻量级，可以启动很多个协程

3. 可以控制






相较于线程，有以下优点

- 最大的优势就是协程极高的执行效率。因为子程序切换不是线程切换，而是**由程序自身控制**，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。
- 第二大优势就是**不需要多线程的锁机制**，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。



 ## 如何创建协程

 ### CoroutineScope类

> Defines a scope for new coroutines. Every coroutine builder is an extension on [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) and inherits its [coroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html) to automatically propagate both context elements and cancellation.

​	定义了所创建的协程的范围。每个协程构建器都继承于它。一般情况下，都是使用launch方法和async方法创建协程。
 - launch方法：

> Launches new coroutine without blocking current thread and returns a reference to the coroutine as a [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html). The coroutine is cancelled when the resulting job is [cancelled](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html).

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {...}
```

使用launch创建协程

```kotlin
import kotlinx.coroutines.*
fun main() {
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L)
        println("World!")
    }
    println("Hello,") // 主线程中的代码会立即执行
    runBlocking {     // 但是这个函数阻塞了主线程
        delay(2000L)  // ……我们延迟2秒来保证 JVM 的存活
    } 
}
```



 - async方法

> Creates new coroutine and returns its future result as an implementation of [Deferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/index.html). The running coroutine is cancelled when the resulting deferred is [cancelled](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html).

```kotlin
public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T>{...}
```

launch 和async 方法参数是一样的：

- `context: CoroutineContext`：表示一个协程的上下文。在Android开发中，可以使用Dispatchers.Main来指定该协程运行在UI线程。下面会详细说明协程上下文和调度器
- `start: CoroutineStart`：定义何时启动协程。默认值是`CoroutineStart.DEFAULT`，表示立即启动协程
- `block: suspend CoroutineScope.() -> T`： 真正的协程并不是launch和async方法，而是`block: suspend CoroutineScope.() -> T`lambda表达式参数。

但是其返回类型不同。launch返回一个Job对象，而async返回一个Deferred对象。Deferred继承了Job接口。Deferred比Job多了一个await方法。

```kotlin
public interface Deferred<out T> : Job {
	public suspend fun await(): T
}
```

launch 一般用于执行协程任务。async用于执行协程任务并得到执行结果。

async 例子

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // 假设我们在这里做了些有用的事
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // 假设我们在这里也做了些有用的事
    return 29
}
```



### runBlocking
协程构建器。创建新的协程，并阻塞当前线程，直到协程执行完毕

```kotlin
@Throws(InterruptedException::class)
public fun <T> runBlocking(context: CoroutineContext = EmptyCoroutineContext, block: suspend CoroutineScope.() -> T): T {
   // ...
}
```

runBlocking方法声明和launch、async非常类似。

## suspend关键字

suspend 修饰的函数或lambda只能在协程中或suspend函数中调用。从上述launch和async方法中第三个参数可知，协程本质也是被suspend修饰的函数。



## 协程上下文

CoroutineContext类

协程上下文包括了一个 *协程调度器* （请参见 [CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html)），它决定了在什么线程中执行协程任务。协程调度器可以将协程的执行局限在指定的线程中，调度它运行在线程池中或让它不受限的运行。在上述文章中，CoroutineContext是协程构建器的参数。在构建协程时，可以通过这个参数，显式的指明协程运行在哪个线程中。

**内置调度器**

- Dispatchers.Main：UI线程

>  A coroutine dispatcher that is confined to the Main thread operating with UI objects. This dispatcher can be used either directly or via [MainScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-main-scope.html) factory. Usually such dispatcher is single-threaded.

- Dispatchers.IO

> The [CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html) that is designed for offloading blocking IO tasks to a shared pool of threads.

- Dispatchers.Default

> The default [CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html) that is used by all standard builders like [launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/index.html#), [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/index.html#), etc if no dispatcher nor any other [ContinuationInterceptor](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-continuation-interceptor/index.html) is specified in their context.

- Dispatchers.Unconfined

> A coroutine dispatcher that is not confined to any specific thread. It executes initial continuation of the coroutine *immediately* in the current call-frame and lets the coroutine resume in whatever thread that is used by the corresponding suspending function, without mandating any specific threading policy. **Note: use with extreme caution, not for general code**.



##  取消与超时

在上文中已说明launch和async的返回类型分别是Job和Deferred。而Deferred是继承于Job接口。在介绍协程的取消操作前，有必要先了解Job接口。

Job类

> A background job. Conceptually, a job is a cancellable thing with a life-cycle that culminates in its completion.

每个任务有以下状态:

| **State**                        | [isActive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/is-active.html) | [isCompleted](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/is-completed.html) | [isCancelled](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/is-cancelled.html) |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| *New* (optional initial state)   | `false`                                                      | `false`                                                      | `false`                                                      |
| *Active* (default initial state) | `true`                                                       | `false`                                                      | `false`                                                      |
| *Completing* (transient state)   | `true`                                                       | `false`                                                      | `false`                                                      |
| *Cancelling* (transient state)   | `false`                                                      | `false`                                                      | `true`                                                       |
| *Cancelled* (final state)        | `false`                                                      | `true`                                                       | `true`                                                       |
| *Completed* (final state)        | `false`                                                      | `true`                                                       | `false`                                                      |

> Usually, a job is created in *active* state (it is created and started). However, coroutine builders that provide an optional `start` parameter create a coroutine in *new* state when this parameter is set to [CoroutineStart.LAZY](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-start/-l-a-z-y.html). Such a job can be made *active* by invoking [start](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/start.html) or [join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html).
>
> A job is *active* while the coroutine is working. Failure of the job with exception makes it *cancelling*. A job can be cancelled it at any time with [cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) function that forces it to transition to *cancelling* state immediately. The job becomes *cancelled* when it finishes executing it work.



Job类常用方法主要是start，cancel，join

- start 方法：如果协程没有启动，则启动协程。如果确实通过这个start启动协程，则返回true。如果协程已启动或已完成，则返回false。

```kotlin
public fun start(): Boolean
```

- cancel方法：取消任务

```kotlin
public fun cancel(): Unit
```

- join方法：Suspends coroutine until this job is complete. This invocation resumes normally (without exception) when the job is complete for any reason and the [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.md) of the invoking coroutine is still [active](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/is-active.html). This function also [starts](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/start.html) the corresponding coroutine if the [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.md) was still in *new* state.

```kotlin
 public suspend fun join()
```





```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) { // 一个执行计算的循环，只是为了占用CPU
            // 每秒打印消息两次
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // 等待一段时间
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // 取消一个任务并且等待它结束
    println("main: Now I can quit.")
}
```



```kotlin
fun test() = runBlocking{
    //每秒输出两个数字
    val job1 = launch(Dispatchers.IO, CoroutineStart.LAZY) {
        var count = 0
        while (true) {
            count++
            //delay()表示将这个协程挂起500ms
            delay(500)
            println("count::$count")
        }
    }

    //job2会立刻启动
    val job2 = async(Dispatchers.IO) {
        job1.start()
        "JOB2"
    }

    launch(Dispatchers.Main) {
        delay(3000)
        job1.cancel()
        //await()的规则是：如果此刻job2已经执行完则立刻返回结果，否则等待job2执行
        println(job2.await())
    }

}
count::1
count::2
count::3
count::4
count::5
count::6
JOB2
```

