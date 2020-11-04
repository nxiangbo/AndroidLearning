# synchronized



synchronized 关键字，可以修饰方法和代码块

对于修饰synchronized（this），synchronized锁住的是括号里的对象，而不是代码。
对于非static的synchronized方法，锁的就是对象本身也就是this。
对于修饰static方法，锁的是Class对象，相当于全局锁，锁住了代码段。
对于修饰synchronized（Sync.class）,锁住的是Class对象，相当于全局锁。

- 工作原理

When a thread wants to execute synchronized statements inside the synchronized block, it MUST acquire the lock on `lockObject`‘s monitor. At a time, only one thread can acquire the monitor of a lock object. So all other threads must wait till this thread, currently acquired the lock, finish it’s execution.

In this way, synchronized keyword guarantees that only one thread will be executing the synchronized block statements at a time, and thus prevent multiple threads from corrupting the shared data inside the block.

Keep in mind that if a thread is put on sleep (using `sleep()` method) then it does not release the lock. At this sleeping time, no thread will be executing the synchronized block statements.

Java synchronization will throw **NullPointerException** if lock object used in `'synchronized (lock)'` is `null`.