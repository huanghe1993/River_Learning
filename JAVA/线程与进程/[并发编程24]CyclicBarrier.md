# CyclicBarrier 

[TOC]

##概述

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。 

CyclicBarrier类似于CountDownLatch也是个计数器， 不同的是CyclicBarrier数的是调用了CyclicBarrier.await()进入等待的线程数， 当线程数达到了CyclicBarrier初始时规定的数目时，所有进入等待状态的线程被唤醒并继续。 CyclicBarrier就象它名字的意思一样，可看成是个障碍， 所有的线程必须到齐后才能一起通过这个障碍。 CyclicBarrier初始时还可带一个Runnable的参数，此Runnable任务在CyclicBarrier的数目达到后，所有其它线程被唤醒前被执行。 



**CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；**

**而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；**



## 方法摘要

| **构造方法摘要**                                             |
| ------------------------------------------------------------ |
| `CyclicBarrier(int parties)`            创建一个新的 `CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在每个 barrier 上执行预定义的操作。 |
| `CyclicBarrier(int parties, Runnable barrierAction)`            创建一个新的 `CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行 |

| **方法摘要** |                                                              |
| ------------ | ------------------------------------------------------------ |
| `int`        | `await**()`            在所有[`参与者`](http://www.cnblogs.com/java/util/concurrent/CyclicBarrier.html#getParties())都已经在此 barrier 上调用 `await` 方法之前，将一直等待。 |
| `int`        | `await**(long timeout, TimeUnit unit)`            在所有[`参与者`](http://www.cnblogs.com/java/util/concurrent/CyclicBarrier.html#getParties())都已经在此屏障上调用 `await` 方法之前，将一直等待。 |
| `int`        | `getNumberWaiting()`            返回当前在屏障处等待的参与者数目。 |
| `int`        | `getParties**()`            返回要求启动此 barrier 的参与者数目。 |
| `boolean`    | `isBroken()`            查询此屏障是否处于损坏状态。         |
| `void`       | `reset()`            将屏障重置为其初始状态。                |

## 代码示例

开会的场景 当所有的人都到会时才开始会议

```java
public class Demo {

    public void meeting(CyclicBarrier barrier) {
        System.out.println(Thread.currentThread().getName() + "到达会议室，等待开会");
        try {
            barrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Demo demo = new Demo();

        CyclicBarrier barrier = new CyclicBarrier(10, new Runnable() {
            @Override
            public void run() {
                System.out.println("开始开会");
            }
        });

        for (int i = 0; i <10 ; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    demo.meeting(barrier);

                }
            }).start();

        }
    }


}
```

