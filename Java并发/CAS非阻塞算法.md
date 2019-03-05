# CAS非阻塞算法
基于CAS的非阻塞算法：一个线程的失败或挂起不应该引起另一个线程的失败或挂起的一种算法。  
一般是利用硬件层面支持的原子化操作指令来取代锁的，比如CAS（compare and swap)，从而保证共享数据在并发访问下的数据一致性。

按照锁的机制划分，一般分为悲观锁和乐观锁。
-  悲观锁
独占锁是一种悲观锁，synchronized就是一种独占锁，会导致其它所有需要锁的线程挂起，等待持有锁的线程释放锁。
-  乐观锁
所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁用到的机制就是CAS，Compare and Swap。
## CAS 
CAS有3个操作数，**内存值V，旧的预期值A，要修改的新值B**。**当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。**

现代的CPU提供了特殊的指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。

CAS和volatile关键字结合可以保证原子性。

以`AtomicInteger`类为例来探究CAS是如何保证数据一致性的。

```java
private volatile int value;
```

首先毫无疑问，在没有锁的机制下可能需要借助volatile原语，保证线程间的数据是可见的（共享的）。

这样才获取变量的值的时候才能直接读取。

```java
public final int get() {
        return value;
    }
```

然后来看看++i是怎么做到的。

```java
public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
```

在这里采用了CAS操作，每次从内存中读取数据然后将此数据和+1后的结果进行CAS操作，如果成功就返回结果，否则重试直到成功为止。

而compareAndSet利用JNI来完成CPU指令的操作。
```java
public final boolean compareAndSet(int expect, int update) {   
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

整体的过程就是这样子的，利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。其它原子操作都是利用类似的特性完成的。

而整个java.util.concurrent都是建立在CAS之上的，因此对于synchronized阻塞算法，java.util.concurrent在性能上有了很大的提升。

### 优点
-  它用硬件的原生形态代替 JVM 的锁定代码路径，从而在更细的粒度层次上（独立的内存位置）进行同步。
-  失败的线程可以立即重试而不用被挂起，降低了争用成本，即使有少量失败的CAS操作，也依然比锁争用造成的重新调度快的多。
-  争用CAS提供更短的延迟（因为争用CAS比争用锁会更快），提供更好的吞吐率。
-  对生存问题（死锁和线程优先级反转）提供更好的防御。

### 缺点
CAS看起来很爽，但是会导致“ABA问题”。

CAS算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。

比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。如果链表的头在变化了两次后恢复了原值，但是不代表链表就没有变化。因此前面提到的原子操作AtomicStampedReference/AtomicMarkableReference就很有用了。这允许一对变化的元素进行原子操作。

