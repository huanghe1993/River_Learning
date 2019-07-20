# Semaphore

[TOC]

## 总述

1、Semaphore就是一个信号量，**它的作用是限制某段代码块的并发数**。

2、Semaphore有一个构造函数，可以**传入一个int型整数n，表示某段代码最多只有n个线程可以访问**，

3、**如果超出了n，那么请等待，等到某个线程执行完毕这段代码块，下一个线程再进入**。

4、由此可以看出如果Semaphore构造函数中传入的int型整数n=1，相当于变成了一个synchronized了。

##方法 

Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

Semaphore类位于java.util.concurrent包下，它提供了2个构造器：

```java
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```

下面说一下Semaphore类中比较重要的几个方法，首先是acquire()、release()方法： 

```java
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

release()用来释放许可。注意，在释放许可之前，必须先获获得许可。

这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

## 示例

假若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现： 

```java
public class Test {  
    public static void main(String[] args) {  
        int N = 8; //工人数  
        Semaphore semaphore = new Semaphore(5); //机器数目  
        for(int i=0;i<N;i++)  
            new Worker(i,semaphore).start();  
    }      
    static class Worker extends Thread{  
        private int num;  
        private Semaphore semaphore;  
        public Worker(int num,Semaphore semaphore){  
            this.num = num;  
            this.semaphore = semaphore;  
        }          
        @Override  
        public void run() {  
            try {  
                semaphore.acquire();  
                System.out.println("工人"+this.num+"占用一个机器在生产...");  
                Thread.sleep(2000);  
                System.out.println("工人"+this.num+"释放出机器");  
                semaphore.release();              
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
} 

```

运行的结果是：

```java
工人0占用一个机器在生产...
工人1占用一个机器在生产...
工人2占用一个机器在生产...
工人3占用一个机器在生产...
工人4占用一个机器在生产...
工人0释放出机器
工人2释放出机器
工人1释放出机器
工人6占用一个机器在生产...
工人7占用一个机器在生产...
工人5占用一个机器在生产...
工人3释放出机器
工人4释放出机器
工人5释放出机器
工人6释放出机器
工人7释放出机器
```

###Executors.newFixedThread（10）的区别

相同点：Executors.newFixedThread和Semaphore相同点是同时可以让固定数量的线程进入

不同点：Executors.newFixedThread（10）是有10个线程，多的线程是创建不出来的，线程执行完之后返回到线程池中，Semaphore是只能让10个线程进入，但是进去的线程不止10个