# Condition

[TOC]

## 前沿

在`java.util.concurrent`包中，有两个很特殊的工具类，`Condition`和`ReentrantLock`，使用过的人都知道，`ReentrantLock`（重入锁）是jdk的`concurrent`包提供的一种独占锁的实现。它继承自Dong Lea的 `AbstractQueuedSynchronizer`（同步器），确切的说是`ReentrantLock`的一个内部类继承了`AbstractQueuedSynchronizer`，`ReentrantLock`只不过是代理了该类的一些方法，可能有人会问为什么要使用内部类在包装一层？ 我想是安全的关系，因为`AbstractQueuedSynchronizer`中有很多方法，还实现了共享锁，`Condition`(稍候再细说)等功能，如果直接使`ReentrantLock`继承它，则很容易出现`AbstractQueuedSynchronizer`中的API被无用的情况。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180718/hGh4LE7ECb.png?imageslim)

## 一、简单示例

`ReentrantLock`和`Condition`的使用方式通常是这样的： 

```java
public class Demo {
	
	private int signal;
	
	public synchronized void a() {
		while(signal != 0 ) {
			try {
				wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		System.out.println("a");
		signal ++;
		notifyAll();
	}
	
	public synchronized void b() {
		while(signal != 1) {
			try {
				wait();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		System.out.println("b");
		signal ++;
		notifyAll();
	}
	
	public synchronized void c () {
		while(signal != 2) {
			try {
				wait();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		System.out.println("c");
		signal = 0;
		notifyAll();
	}
	
	public static void main(String[] args) {
		
		Demo d = new Demo();
		A a = new A(d);
		B b = new B(d);
		C c = new C(d);
		
		new Thread(a).start();
		new Thread(b).start();
		new Thread(c).start();
		
	}
}

class A implements Runnable {
	
	private Demo demo;
	
	public A(Demo demo) {
		this.demo = demo;
	}

	@Override
	public void run() {
		while(true) {
			demo.a();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}
class B implements Runnable {
	
	private Demo demo;
	
	public B(Demo demo) {
		this.demo = demo;
	}
	
	@Override
	public void run() {
		while(true) {
			demo.b();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}
class C implements Runnable {
	
	private Demo demo;
	
	public C(Demo demo) {
		this.demo = demo;
	}
	
	@Override
	public void run() {
		while(true) {
			demo.c();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}
```

运行后，结果如下：

a,b,c按照顺序进行执行

使用Condition的方式改写上面的代码：

```java

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Demo {
	
	private int signal;
	
	Lock lock = new ReentrantLock();
	Condition a = lock.newCondition();
	Condition b = lock.newCondition();
	Condition c = lock.newCondition();
	
	
	public void a() {
		lock.lock();
		while(signal != 0 ) {
			try {
				a.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		System.out.println("a");
		signal ++;
		b.signal();
		lock.unlock();
	}
	
	public  void b() {
		lock.lock();
		while(signal != 1) {
			try {
				b.await();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		System.out.println("b");
		signal ++;
		c.signal();
		lock.unlock();
	}
	
	public  void c () {
		lock.lock();
		while(signal != 2) {
			try {
				c.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		System.out.println("c");
		signal = 0;
		a.signal();
		lock.unlock();
	}
	
	public static void main(String[] args) {
		
		Demo d = new Demo();
		A a = new A(d);
		B b = new B(d);
		C c = new C(d);
		
		new Thread(a).start();
		new Thread(b).start();
		new Thread(c).start();
		
	}
}

class A implements Runnable {
	
	private Demo demo;
	
	public A(Demo demo) {
		this.demo = demo;
	}

	@Override
	public void run() {
		while(true) {
			demo.a();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}
class B implements Runnable {
	
	private Demo demo;
	
	public B(Demo demo) {
		this.demo = demo;
	}
	
	@Override
	public void run() {
		while(true) {
			demo.b();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}
class C implements Runnable {
	
	private Demo demo;
	
	public C(Demo demo) {
		this.demo = demo;
	}
	
	@Override
	public void run() {
		while(true) {
			demo.c();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}
```

## 生成者消费者模型使用Condition

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Tmall2 {

	private int count;

	private Lock lock = new ReentrantLock();
	Condition p = lock.newCondition();
	Condition t = lock.newCondition();

	public final int MAX_COUNT = 10;

	public void push() {
		lock.lock();
		while (count >= MAX_COUNT) {
			try {
				System.out.println(Thread.currentThread().getName() + " 库存数量达到上限，生产者停止生产。");
				p.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		count++;
		System.out.println(Thread.currentThread().getName() + " 生产者生产，当前库存为：" + count);
		t.signal();
		lock.unlock();
	}

	public void take() {
		lock.lock();
		while (count <= 0) {
			try {
				System.out.println(Thread.currentThread().getName() + " 库存数量为零，消费者等待。");
				t.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		count--;
		System.out.println(Thread.currentThread().getName() + " 消费者消费，当前库存为：" + count);
		p.signal();
		lock.unlock();
	}

}

```

## Condition实现有界的队列

```java
package com.roocon.thread.tb1;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class MyQueue<E> {

	private Object[] obj;

	private int addIndex;
	private int removeIndex;
	private int queueSize;

	private Lock lock = new ReentrantLock();
	Condition addCondition = lock.newCondition();
	Condition removeCondition = lock.newCondition();

	public MyQueue(int count) {
		obj = new Object[count];
	}

	public void add(E e) {
		lock.lock();
		while (queueSize == obj.length) {
			try {
				addCondition.await();
			} catch (InterruptedException e1) {
				e1.printStackTrace();
			}
		}
		obj[addIndex] = e;

		if (++addIndex == obj.length) {
			addIndex = 0;
		}

		queueSize++;
		removeCondition.signal();
		lock.unlock();
	}

	public void remove() {
		lock.lock();

		while (queueSize == 0) {
			try {
				removeCondition.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		obj[removeIndex] = null;

		if (++removeIndex == obj.length) {
			removeIndex = 0;
		}

		queueSize--;

		addCondition.signal();

		lock.unlock();
	}
	
}

```

## Condition实现原理

首先还是要明白，`reentrantLock.newCondition()` 返回的是`Condition`的一个实现，该类在`AbstractQueuedSynchronizer`中被实现，叫做`newCondition()` ；

```java
public Condition newCondition()   { return sync.newCondition(); }
```

它可以访问`AbstractQueuedSynchronizer`中的方法和其余内部类（`AbstractQueuedSynchronizer`是个抽象类，至于他怎么能访问，这里有个很奇妙的点，后面我专门用demo说明 ） 

当`await`被调用时，代码如下： 

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    // 将当前线程包装下后，添加到Condition自己维护的一个链表中。等待队列
    Node node = addConditionWaiter(); 
    // 释放当前线程占有的锁，从demo中看到，// 调用await前，当前线程是占有锁的                            
    int savedState = fullyRelease(node);                                       
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {// 释放完毕后，遍历AQS的队列，看当前节点是否在队列中，
        // 不在 说明它还没有竞争锁的资格，所以继续将自己沉睡。
        // 直到它被加入到队列中，聪明的你可能猜到了，
        // 没有错，在singal的时候加入不就可以了？
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // 被唤醒后，重新开始正式竞争锁，同样，如果竞争不到还是会将自己沉睡，等待唤醒重新开始竞争。
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

1. 线程1调用`reentrantLock.lock`时，线程被加入到AQS的等待队列中。
2. 线程1调用`await`方法被调用时，该线程从AQS中移除，对应操作是锁的释放。
3. 接着马上被加入到`Condition`的等待队列中，以为着该线程需要`signal`信号。
4. 线程2，因为线程1释放锁的关系，被唤醒，并判断可以获取锁，于是线程2获取锁，并被加入到AQS的等待队列中。
5. 线程2调用`signal`方法，这个时候`Condition`的等待队列中只有线程1一个节点，于是它被取出来，并被加入到AQS的等待队列中。 注意，这个时候，线程1 并没有被唤醒。
6. `signal`方法执行完毕，线程2调用`reentrantLock.unLock()`方法，释放锁。这个时候因为AQS中只有线程1，于是，AQS释放锁后按从头到尾的顺序唤醒线程时，线程1被唤醒，于是线程1回复执行。
7. 直到释放所整个过程执行完毕。

可以看到，整个协作过程是靠结点在AQS的等待队列和`Condition`的等待队列中来回移动实现的，`Condition`作为一个条件类，很好的自己维护了一个等待信号的队列，并在适时的时候将结点加入到AQS的等待队列中来实现的唤醒操作。 

## 同步队列和等待队列

Synchronized：就是实现的是同步队列

await：实现的就是等待队列，每一个Condition就是等待队列