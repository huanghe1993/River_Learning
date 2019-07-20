# CountDownLatch

[TOC]

## 总述

CountDownLatch 是Java的concurrent包里面的一个计数器，只不过这个计数器的操作是原子操作，同时只能有一个线程去操作这个计数器，也就是同时只能有一个线程去减这个计数器里面的值。在一个线程中如果调用了await()方法，这个线程就会进入到等待的状态，当计数器的值为0的时候这个线程才继续执行



### 1、CountDownLatch是什么 

CountDownLatch可以控制线程的执行，他可以让所有持有他的多个线程同时执行，也可以控制单个线程执行。

他初始化的时候会传出一个int类型的参数i，调用一次countDown(）方法后i的值会减1。

在一个线程中如果调用了await()方法，这个线程就会进入到等待的状态，当参数i为0的时候这个线程才继续执行。

### 2、一个简单的跑步比赛分析

比如一个跑步比赛，有五个选手参加，有两点需要注意，第一我们必须确保这5个选手都准备就绪了，才能宣布比赛开始，第二只有当5个选手都完成比赛了才能宣布比赛结束。

所以假设这5个选手都有一个独立的线程进行跑步，那么每个线程都必须持有一个相同的CountDownLatch，并且调用await()进入等待，然后才能确保他们同事开始比赛。

同时可以主线程中也要持有一个CountDownLatch，当选手开始比赛后要进入到等待状态，等待所有的选手都完成比赛才能宣布比赛结束。

###3、主要方法

 public CountDownLatch(int count);

 public void **countDown**();

 public void **await**() throws [InterruptedException](http://zk1878.iteye.com/java/lang/InterruptedException.html)

构造方法参数指定了计数的次数

countDown方法，当前线程调用此方法，则计数减一

awaint方法，调用此方法会一直阻塞当前线程，直到计时器的值为0

### 4、示例

```java
package Thread4;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @Author: River
 * @Date:Created in  21:52 2018/8/8
 * @Description:
 */
public class Demo {

    private int[] nums;

    public Demo(int nums) {
        this.nums =new int[nums];
    }

    public void calc(String line, int index) {
        String[] nus = line.split(",");
        int total = 0;
        for (String num : nus) {
            total += Integer.valueOf(num);
        }
        nums[index] = total;
        System.out.println(Thread.currentThread().getName() + "计算任务。。" + line+"结果为："+total);
    }

    public void sum() {
        System.out.println("汇总线程开始执行");
        int total = 0;
        for (int i = 0; i < nums.length; i++) {
            total += nums[i];
        }
        System.out.println("最终的结果是："+total);
    }

    public static void main(String[] args) throws IOException {

        List<String> content = readFile();
        int size = content.size();
        Demo demo = new Demo(size);

        //创建线程任务，启动线程
        for (  int  i = 0; i < size; i++) {
            final int j = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    demo.calc(content.get(j), j);
                }
            }).start();
        }

        //空等待
        while (Thread.activeCount() > 1) {

        }
        demo.sum();

    }

    private static List<String> readFile() throws IOException {
        List<String> contents = new ArrayList<>();
        String line = null;
        BufferedReader reader = new BufferedReader(new FileReader("E:\\test.txt"));
        while ((line = reader.readLine()) != null) {
            contents.add(line);
        }
        return contents;
    }
}

```

利用CountDownLatch来优化空等待

```java
package Thread4;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;

/**
 * @Author: River
 * @Date:Created in  21:52 2018/8/8
 * @Description:
 */
public class Demo {

    private int[] nums;

    public Demo(int nums) {
        this.nums =new int[nums];
    }

    public void calc(String line, int index ,CountDownLatch latch) {
        String[] nus = line.split(",");
        int total = 0;
        for (String num : nus) {
            total += Integer.valueOf(num);
        }
        nums[index] = total;
        System.out.println(Thread.currentThread().getName() + "计算任务。。" + line+"结果为："+total);
    }

    public void sum() {
        System.out.println("汇总线程开始执行");
        int total = 0;
        for (int i = 0; i < nums.length; i++) {
            total += nums[i];
        }
        System.out.println("最终的结果是："+total);
    }

    public static void main(String[] args) throws IOException {

        List<String> content = readFile();
        int size = content.size();

        CountDownLatch latch = new CountDownLatch(size);

        Demo demo = new Demo(size);

        //创建线程任务，启动线程
        for (  int  i = 0; i < size; i++) {
            final int j = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    demo.calc(content.get(j), j,latch);
                }
            }).start();
        }

        //空等待
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        demo.sum();

    }

    private static List<String> readFile() throws IOException {
        List<String> contents = new ArrayList<>();
        String line = null;
        BufferedReader reader = new BufferedReader(new FileReader("E:\\test.txt"));
        while ((line = reader.readLine()) != null) {
            contents.add(line);
        }
        return contents;

    }


}

```

### 更加简单的例子

```java
public class Test {
    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(2);
        new Thread() {
            public void run() {
                try {
                    System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程" + Thread.currentThread().getName() + "执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();

        new Thread() {
            public void run() {
                try {
                    System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程" + Thread.currentThread().getName() + "执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
        try {
            System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {            e.printStackTrace();        }    }}


```

执行的结果是：

```xml
线程Thread-0正在执行 线程Thread-1正在执行 等待2个子线程执行完毕... 线程Thread-0执行完毕 线程Thread-1执行完毕 2个子线程已经执行完毕 继续执行主线程
```

