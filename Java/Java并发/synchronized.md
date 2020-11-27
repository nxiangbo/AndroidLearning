# synchronized



## 基本使用

synchronized 关键字，可以修饰方法和代码块

对于修饰synchronized（this），synchronized锁住的是括号里的对象，而不是代码。
对于非static的synchronized方法，锁的就是对象本身也就是this。
对于修饰static方法，锁的是Class对象，相当于全局锁，锁住了代码段。
对于修饰synchronized（Sync.class）,锁住的是Class对象，相当于全局锁。

当一个线程执行synchronized语句块时，必须先获取对象的monitor锁。一次仅有一个线程可以获取对象的monitor锁。因此，其他线程需要等待锁被释放。

当线程sleep时，并不会释放它持有的锁。此时，没有线程执行synchronized代码块。

如果`synchronized (lock)`中lock对象为null，会抛出空指针异常。



## 实现原理

synchronized 是可重入锁
moniter 指令，线程持有锁后，加1，代码块执行完，减1。等于0时，表示锁被释放。

如果对上面的执行结果还有疑问，也先不用急，我们先来了解Synchronized的原理，再回头上面的问题就一目了然了。我们先通过反编译下面的代码来看看Synchronized是如何实现对代码块进行同步的：



```java
1 package com.paddx.test.concurrent;
2 
3 public class SynchronizedDemo {
4     public void method() {
5         synchronized (this) {
6             System.out.println("Method 1 start");
7         }
8     }
9 }
```



反编译结果：

![img](https://images2015.cnblogs.com/blog/820406/201604/820406-20160414215316020-1963237484.png)

关于这两条指令的作用，我们直接参考JVM规范中描述：

monitorenter ：

> Each object is associated with a monitor. A monitor is locked if and only if it has an owner. The thread that executes monitorenter attempts to gain ownership of the monitor associated with objectref, as follows:
> • If the entry count of the monitor associated with objectref is zero, the thread enters the monitor and sets its entry count to one. The thread is then the owner of the monitor.
> • If the thread already owns the monitor associated with objectref, it reenters the monitor, incrementing its entry count.
> • If another thread already owns the monitor associated with objectref, the thread blocks until the monitor's entry count is zero, then tries again to gain ownership.

这段话的大概意思为：

每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

3.如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

monitorexit：　

> The thread that executes monitorexit must be the owner of the monitor associated with the instance referenced by objectref.
> The thread decrements the entry count of the monitor associated with objectref. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so.

这段话的大概意思为：

执行monitorexit的线程必须是objectref所对应的monitor的所有者。

指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。 

　　通过这两段描述，我们应该能很清楚的看出Synchronized的实现原理，Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

　　我们再来看一下同步方法的反编译结果：

源代码：



```java
1 package com.paddx.test.concurrent;
2 
3 public class SynchronizedMethod {
4     public synchronized void method() {
5         System.out.println("Hello World!");
6     }
7 }
```

反编译结果：

![img](https://images2015.cnblogs.com/blog/820406/201604/820406-20160418202553429-1642545018.png)

　　从反编译的结果来看，方法的同步并没有通过指令monitorenter和monitorexit来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。