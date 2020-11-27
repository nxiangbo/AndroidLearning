# 闭锁 CountDownLatch

## 什么是CountDownLatch

闭锁（Latch）：一种同步方法，可以延迟线程的进度直到线程到达某个终点状态。通俗的讲就是，一个闭锁相当于一扇大门，在大门打开之前所有线程都被阻断，一旦大门打开所有线程都将通过，但是一旦大门打开，所有线程都通过了，那么这个闭锁的状态就失效了，门的状态也就不能变了，只能是打开状态。也就是说闭锁的状态是一次性的，它确保在闭锁打开之前所有特定的活动都需要在闭锁打开之后才能完成。

CountDownLatch是JDK 5+里面闭锁的一个实现，允许一个或者多个线程等待某个事件的发生。CountDownLatch有一个正数计数器，countDown方法对计数器做减操作，await方法等待计数器达到0。所有await的线程都会阻塞直到计数器为0或者等待线程中断或者超时。



假如有主线程需要等待线程A和线程B的任务执行完成，才能继续执行。

```java
package concurrent;

import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {

    static CountDownLatch latch;

    static class WorkerThread extends Thread {
        private String name;
        public WorkerThread(String name) {
            this.name = name;
        }
        @Override
        public void run() {
            super.run();

            System.out.println("执行任务：" + name + " thread = " + Thread.currentThread().getName());

            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
        }
    }

    public static void main(String[] args) {
        latch = new CountDownLatch(2);
        WorkerThread A = new WorkerThread("A");
        WorkerThread B = new WorkerThread("B");
        A.start();
        B.start();
        
        System.out.println("执行主线程任务 thread = " + Thread.currentThread().getName() );


    }
}

```



## CountDownLatch实现原理

