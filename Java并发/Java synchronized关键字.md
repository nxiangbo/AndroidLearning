# Java synchronized关键字
对于修饰synchronized（this），synchronized锁住的是括号里的对象，而不是代码。
对于非static的synchronized方法，锁的就是对象本身也就是this。
对于修饰static方法，锁的是Class对象，相当于全局锁，锁住了代码段。
对于修饰synchronized（Sync.class）,锁住的是Class对象，相当于全局锁。

