# 阻塞队列BlockingQueue

[TOC]

在Java多线程应用中，队列的使用率很高，多数生产消费模型的首选数据结构就是队列(先进先出)。Java提供的线程安全的Queue可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是BlockingQueue，非阻塞队列的典型例子是ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。

**lockingQueue 是一个先进先出的队列（Queue），为什么说是阻塞（Blocking）的呢？**是因为 BlockingQueue 支持当获取队列元素但是队列为空时，会阻塞等待队列中有元素再返回；也支持添加元素时，如果队列已满，那么等到队列可以放入新元素时再放入。 

### ArrayBlockingQueue

ArrayBlockingQueue是一个用数组实现的有界阻塞队列。此队列按照先进先出（FIFO）的原则对元素进行排序。默认情况下不保证访问者公平的访问队列，所谓公平访问队列是指阻塞的所有生产者线程或消费者线程，当队列可用时，可以按照阻塞的先后顺序访问队列，即先阻塞的生产者线程，可以先往队列里插入元素，先阻塞的消费者线程，可以先从队列里获取元素。通常情况下为了保证公平性会降低吞吐量。我们可以使用以下代码创建一个公平的阻塞队列： 

```java
ArrayBlockingQueue fairQueue = new  ArrayBlockingQueue(1000,true);
```

 ### 生产者消费者模型来改进

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。 

阻塞队列提供了四种处理方法:

| 方法\处理方式 | 抛出异常  | 返回特殊值 | 一直阻塞 | 超时退出           |
| ------------- | --------- | ---------- | -------- | ------------------ |
| 插入方法      | add(e)    | offer(e)   | put(e)   | offer(e,time,unit) |
| 移除方法      | remove()  | poll()     | take()   | poll(time,unit)    |
| 检查方法      | element() | peek()     | 不可用   | 不可用             |

- 抛出异常：是指当阻塞队列满时候，再往队列里插入元素，会抛出IllegalStateException("Queue full")异常。当队列为空时，从队列里获取元素时会抛出NoSuchElementException异常 。
- 返回特殊值：插入方法会返回是否成功，成功则返回true。移除方法，则是从队列里拿出一个元素，如果没有则返回null
- 一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里take元素，队列也会阻塞消费者线程，直到队列可用。
- 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出。
- JDK7提供了7个阻塞队列。分别是
  - ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
  - LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
  - PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
  - DelayQueue：一个使用优先级队列实现的无界阻塞队列。
  - SynchronousQueue：一个不存储元素的阻塞队列。
  - LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
  - LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

定义一个接口Shop：

```java
public interface Shop {
	
	public void push ();
	
	public void take ();

	public void size() ;
}

```

实现方式Tmall3：

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class Tmall3 implements Shop {

	public final int MAX_COUNT = 10;

	private BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(MAX_COUNT);

	public void push() {
		try {
			queue.put(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	public void take() {
		try {
			queue.take();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	public void size() {
		while (true) {
			System.out.println("当前队列的长队为：" + queue.size());
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}

```

生产者PushTarget.java：

```java
public class PushTarget implements Runnable {

	private Shop tmall;
	
	public PushTarget(Shop tmall) {
		this.tmall = tmall;
	}
	
	@Override
	public void run() {
		while(true) {
			tmall.push();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}
```

消费者TakeTarget：

```java
public class TakeTarget implements Runnable {
	
	private Shop tmall;
	
	public TakeTarget(Shop tmall) {
		this.tmall = tmall;
	}

	@Override
	public void run() {
		while(true) {
			tmall.take();
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}
```

测试代码：

```java
public class Main {
	
	public static void main(String[] args) {
		
		Shop tmall = new Tmall3();
		
		PushTarget p = new PushTarget(tmall);
		TakeTarget t = new TakeTarget(tmall);
		
		new Thread(p).start();
		new Thread(p).start();
		new Thread(p).start();
		new Thread(p).start();
		new Thread(p).start();
		new Thread(p).start();
		new Thread(p).start();
		new Thread(p).start();
		
		new Thread(t).start();
		new Thread(t).start();
		new Thread(t).start();
		new Thread(t).start();
		new Thread(t).start();
		new Thread(t).start();
		new Thread(t).start();
		new Thread(t).start();
		
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				tmall.size();				
			}
		}).start();
	}
	
}
```

### 简单消息队列

点对点模式 

发布-订阅模式

**消息队列的应用场景：**

采集数据通过传感器收集(高速的数据) ———>RabbitMq———>java服务端从消息队列里面取数据

