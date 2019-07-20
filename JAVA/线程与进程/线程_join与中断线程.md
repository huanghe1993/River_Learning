## join与中断线程（2）

[TOC]



##1、join

```
join()方法：加入线程让调用的线程先执行指定时间或者是指向完毕

异常：InterruptedException-如果任何线程中断当前线程。当抛出异常时，当前线程的中断状态被清除
```

```java
package com.huanghe;

/**
 * @Author: River
 * @Date:Created in  10:27 2018/5/28
 * @Description: join()方法：加入线程让调用的线程先执行指定时间或者是指向完毕
 */
public class ThreadDemo2 {
    public static void main(String[] args) {

        MyRunable2 mr2 = new MyRunable2();
        Thread t = new Thread(mr2);
        t.start();

        for (int i = 0; i <20 ; i++) {
            System.out.println(Thread.currentThread().getName()+"--"+i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (i == 10) {
                try {
                    t.join();//让t线程执行完毕
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }

    }
}

class MyRunable2 implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i <20 ; i++) {
            System.out.println(Thread.currentThread().getName()+"--"+i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
执行的结果如下：
main--0
Thread-0--0
main--1
Thread-0--1
main--2
Thread-0--2
main--3
Thread-0--3
Thread-0--4
main--4
Thread-0--5
main--5
Thread-0--6
main--6
Thread-0--7
main--7
main--8
Thread-0--8
Thread-0--9
main--9
main--10
Thread-0--10
Thread-0--11
Thread-0--12
Thread-0--13
Thread-0--14
Thread-0--15
Thread-0--16
Thread-0--17
Thread-0--18
Thread-0--19
main--11
main--12
main--13
main--14
main--15
main--16
main--17
main--18
main--19

```

## 2、中断

```java
public void interrupt():
//中断这个线程
//除非当前线程中断自身，这是始终允许的
```

```java
public static boolean interrupted()
 //测试当前线程是否中断。该方法可以清除中断状态。换句话说，如果这个方法被连续的调用两次，那么第二个调用将返回false(除非当前线程再次中断，在第一个调用已经清除其中的中断状态之后，在第二个调用之前已经检查过)
```

```java
package com.huanghe;

/**
 * @Author: River
 * @Date:Created in  10:27 2018/5/28
 * @Description: join()方法：加入线程让调用的线程先执行指定时间或者是指向完毕
 */
public class ThreadDemo2 {
    public static void main(String[] args) {

        MyRunable2 mr2 = new MyRunable2();
        Thread t = new Thread(mr2);
        t.start();

        for (int i = 0; i <20 ; i++) {
            System.out.println(Thread.currentThread().getName()+"--"+i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
           if (i == 10) {
//                try {
//                    t.join();//让t线程执行完毕
//                }catch (InterruptedException e){
//                    e.printStackTrace();
//                }
               t.interrupt();//1、中断线程,中断标记
            }
        }

    }
}

class MyRunable2 implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i <20 ; i++) {
             if (Thread.interrupted()){//4）测试中断状态，此方法会把中断标记清除
                break;
            }
            System.out.println(Thread.currentThread().getName()+"--"+i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace(); //2、因为在sleep中会抛出中断异常，并且异常会被清除
                 t.interrupt();//3、中断标记需要再次的被打上
            }
        }
    }
}
//执行的结果
main--0
Thread-0--0
main--1
java.lang.InterruptedException: sleep interrupted
Thread-0--1
	at java.lang.Thread.sleep(Native Method)
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
java.lang.InterruptedException: sleep interrupted
main--2
Thread-0--2
	at java.lang.Thread.sleep(Native Method)
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
main--3
java.lang.InterruptedException: sleep interrupted
Thread-0--3
	at java.lang.Thread.sleep(Native Method)
Thread-0--4
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
main--4
java.lang.InterruptedException: sleep interrupted
Thread-0--5
	at java.lang.Thread.sleep(Native Method)
Thread-0--6
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
Thread-0--7
main--5
java.lang.InterruptedException: sleep interrupted
Thread-0--8
	at java.lang.Thread.sleep(Native Method)
main--6
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
Thread-0--9
	at java.lang.Thread.run(Thread.java:745)
java.lang.InterruptedException: sleep interrupted
main--7
	at java.lang.Thread.sleep(Native Method)
Thread-0--10
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
Thread-0--11
	at java.lang.Thread.run(Thread.java:745)
java.lang.InterruptedException: sleep interrupted
main--8
	at java.lang.Thread.sleep(Native Method)
Thread-0--12
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
java.lang.InterruptedException: sleep interrupted
Thread-0--13
	at java.lang.Thread.sleep(Native Method)
main--9
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
Thread-0--14
	at java.lang.Thread.run(Thread.java:745)
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
Thread-0--15
main--10
Thread-0--16
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
main--11
Thread-0--17
main--12
java.lang.InterruptedException: sleep interrupted
Thread-0--18
	at java.lang.Thread.sleep(Native Method)
Thread-0--19
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
main--13
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.huanghe.MyRunable2.run(ThreadDemo2.java:42)
	at java.lang.Thread.run(Thread.java:745)
main--14
main--15
main--16
main--17
main--18
main--19

```

从执行的结果中可以看出，程序并没有停止执行，但是抛出了异常，这个异常是Runable里面的sleep抛出的异常，t.interrup();只是做了一个中断标记，t是不是真的需要中断进行，需要线程自己进行决定；

所以如果需要完成一个正常的中断，需要上面的四步才可以顺序的完成

## 3、自定义标记中断线程

由于上面的中断过程很麻烦，所以在做中断线程的时候使用的是自定义中断线程

```java
package com.huanghe;

/**
 * @Author: River
 * @Date:Created in  10:27 2018/5/28
 * @Description: join()方法：加入线程让调用的线程先执行指定时间或者是指向完毕
 * (1):使用Interrupt方法中断线程，设置一个中断标记
 * (2):自定义中断标记
 */
public class ThreadDemo2 {
    public static void main(String[] args) {

        MyRunable2 mr2 = new MyRunable2();
        Thread t = new Thread(mr2);
//        t.start();

        MyRunable3 mr3 = new MyRunable3();
        Thread t2 = new Thread(mr3);
        t2.start();

        for (int i = 0; i <20 ; i++) {
            System.out.println(Thread.currentThread().getName()+"--"+i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
           if (i == 10) {
//                try {
//                    t.join();//让t线程执行完毕
//                }catch (InterruptedException e){
//                    e.printStackTrace();
//                }
//               t.interrupt();//中断线程,中断标记
               mr3.flag=false;//自定义中断异常
            }

        }

    }
}

class MyRunable2 implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i <20 ; i++) {
            if (Thread.interrupted()){//测试中断状态，此方法会把中断标记清除
                break;
            }
            System.out.println(Thread.currentThread().getName()+"--"+i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
                Thread.currentThread().interrupt();
            }
        }
    }
}


class MyRunable3 implements Runnable{

    public boolean flag=true;

    public MyRunable3(){
        flag = true;
    }

    @Override
    public void run() {
        int i = 0;
        while (flag) {
            System.out.println(Thread.currentThread().getName() + "====" + (i++));
            try {
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}
```

