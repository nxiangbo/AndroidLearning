# wait vs sleep

A [`wait`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#wait()) can be "woken up" by another thread calling [`notify`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#notify()) on the monitor which is being waited on whereas a [`sleep`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html#sleep(long)) cannot. Also a `wait` (and `notify`) must happen in a block `synchronized` on the monitor object whereas `sleep` does not:

```java
Object mon = ...;
synchronized (mon) {
    mon.wait();
} 
```

At this point the currently executing thread waits *and releases the monitor*. Another thread may do

```java
synchronized (mon) { mon.notify(); }
```

(on the same `mon` object) and the first thread (assuming it is the only thread waiting on the monitor) will wake up.

You can also call [`notifyAll`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#notifyAll()) if more than one thread is waiting on the monitor – this will wake *all of them up*. However, only one of the threads will be able to grab the monitor (remember that the `wait` is in a `synchronized` block) and carry on – the others will then be blocked until they can acquire the monitor's lock.

Another point is that you call `wait` on [`Object`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html) itself (i.e. you wait on an object's monitor) whereas you call `sleep` on [`Thread`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html).

Yet another point is that you can get *spurious wakeups* from `wait` (i.e. the thread which is waiting resumes for no apparent reason). You should **always `wait` whilst spinning on some condition** as follows:

```java
synchronized {
    while (!condition) { mon.wait(); }
}
```