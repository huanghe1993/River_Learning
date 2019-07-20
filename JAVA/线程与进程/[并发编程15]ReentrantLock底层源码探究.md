# ReentrantLock源码探究

[TOC]

## 1、ReentrantLock介绍

> A reentrant mutual exclusion Lock with the same basic behavior and semantics as the implicit monitor lock accessed using synchronized methods and statements, but with extended capabilities.

简单的说ReentrantLock是一种扩展的synchronized可重入互斥锁。主要有一下几个特性

- 一个ReentrantLock总是被当前获得锁的线程所拥有
- ReentrantLock支持公平锁与非公平锁(tryLock方法例外)
- 如其他内置锁表现相同，反序列化的对象总是非锁定的
- 支持同一线程最大数为2147483647的递归获取锁

## 2、Sync继承结构

由于ReentrantLock内部方法执行时基本上都将调用Sync的相应方法去完成，即使用Sync类来作为基础的锁的同步控制，所以Sync是ReentrantLock的核心对象。所以先简单看下Sync类的结构，如下图

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180718/26AI8l648f.png?imageslim)

## 3、基本方法分析

### 3.1 构造方法

```java
    public ReentrantLock() {
        sync = new NonfairSync();//Sync的子类
    }
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

默认的构造方法是非公平锁(文档指出这样的效率相对更加高),同时也可根据参数来创建公平锁。

### 3.2 lock方法

```java
    //非公平锁的lock方法
    final void lock() {
        //lock()方法先通过CAS尝试将状态state从0修改为1(独占锁：1代表已占有，0代表未占有)。若直接修           改成功，前提条件自然是锁的状态为0，则直接将线程的OWNER修改为当前线程，这是一种理想情况，如           果并发粒度设置适当也是一种乐观情况
        if (compareAndSetState(0, 1)){
        //CAS状态赋值，如果成功表示当前锁没有被获取、设置当前独占模式线程获得锁
            setExclusiveOwnerThread(Thread.currentThread());
        }
        else{
            acquire(1);//请求锁(AQS的acquire方法)
        }
    }

    public final void acquire(int arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)){
        //如果获取失败、线程等待时获取锁成功且在等待时被中断
            selfInterrupt();//调用线程interrupt方法，清除中断标记
        }
    }
```

由上可见非公平锁获取锁，首先优先利用CAS机制根据当前状态来获取锁，其次通过acquire方式获取锁，利用acquire方式获取锁又分为两步-tryAcquire、acquireQueued。先看tryAcquire方法如何工作

```java
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }

    final boolean nonfairTryAcquire(int acquires) {
        //获取当前的线程
        final Thread current = Thread.currentThread();
        //得到线程的状态
        int c = getState();
        //判断是不是锁还没有被线程占有state =0 代表的是已占有，state=1代表的是已经占有
        if (c == 0) {
        //当前锁未被获取，尝试CAS机制先置状态位、然后获取锁
            if (compareAndSetState(0, acquires)) {//设置线程的状态为1
                //设置拥有该独占锁的线程为该线程
                setExclusiveOwnerThread(current);
                //返回true就可以获取到锁
                return true;
            }
            
        }
        //（可重入的条件）置状态位失败，说明有其他线程获得锁，先判断是不是当前的线程，如果是当前线程           ，线程的状态位+1，并且线程不会被锁在外面，返回true
        else if (current == getExclusiveOwnerThread()) {//当前线程再进来相当于是重入了
            //状态加1
            int nextc = c + acquires;
            if (nextc < 0) // 重复获取次数溢出,这里有个问题是，为什么是重复的次数太多了会溢出
                throw new Error("Maximum lock count exceeded");
            //更新状态
            setState(nextc);
            return true;
        }
        //如果线程状态不为0，或者是进来的不是当前线程就返回fasle
        return false;
    }
```

上面的过程如果成功就拿到了锁，如果不成功就要调用` acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`我们接下来就应该去研究一下addWaiter()和acquireQueued()

```java
/**
* 【AQS的代码】
* Node结点的数据类型：pre :前节点 next：后一个结点  thread:线程对象  waitStatus:等待状态
* CANCELLED =  1;//由于超时或中断，节点已被取消
*  SIGNAL   = -1;//表示下一个节点是通过park堵塞的，需要通过unpark唤醒
* CONDITION = -2;//表示线程在等待条件变量（先获取锁，加入到条件等待队列，然后释放锁，等待条件变量满* * 足条件；只有重新获取锁之后才能返回）
* 
*/

private Node addWaiter(Node mode) {
		// 1. 将当前线程构建成Node类型 ,EXCLUSIVE 标识等待节点处于独占模式
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        // 2. 当前尾节点是否为null？
		Node pred = tail;
        //如果当前尾结点不为空，说明此时不是空队列
        if (pred != null) {
			// 2.2 将当前节点尾插入的方式插入同步队列中
            node.prev = pred;
            //当前的尾结点进行更新，并返回新加入的结点
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
		// 2.1. 当前同步队列尾节点为null，说明当前线程是第一个加入同步队列进行等待的线程
        enq(node);
        return node;
}
```

分析可以看上面的注释。程序的逻辑主要分为两个部分：

1. 当前同步队列的尾节点为null，调用方法enq()插入;

2. 当前队列的尾节点不为null，则采用尾插入（compareAndSetTail（）方法）的方式入队。

3. **另外还会有另外一个问题：如果 `if (compareAndSetTail(pred, node))`为false怎么办？会继续执行到enq()方法，同时很明显compareAndSetTail是一个CAS操作，通常来说如果CAS操作失败会继续自旋（死循环）进行重试**

因此，经过我们这样的分析，enq()方法可能承担两个任务：

**1.处理当前同步队列尾节点为null时进行入队操作；**

**2. 如果CAS尾插入节点失败后负责自旋进行尝试。**

那么是不是真的就像我们分析的一样了？只有源码会告诉我们答案,enq()源码如下：

```java
/**【AQS的代码】
*
*/
private Node enq(final Node node) {
        for (;;) {
            //获取当前的尾结点
            Node t = tail;
            //如果尾结点为空，也就是enq()的第一种，同步队列为空
			if (t == null) { // Must initialize
				//1. 构造头结点
                if (compareAndSetHead(new Node()))
                    //尾结点和头结点指向的是同一个结点
                    tail = head;
            } else {
				// 2. 尾插入，CAS操作失败自旋尝试
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
}
```

在上面的分析中我们可以看出在第1步中会先创建头结点，说明同步队列是**带头结点的链式存储结构**。带头结点与不带头结点相比，会在入队和出队的操作中获得更大的便捷性，因此同步队列选择了带头结点的链式存储结构。那么带头节点的队列初始化时机是什么？自然而然是在**tail为null时，即当前线程是第一次插入同步队列**。compareAndSetTail(t, node)方法会利用CAS操作设置尾节点，如果CAS操作失败会在`for (;;)`for死循环中不断尝试，直至成功return返回为止。因此，对enq()方法可以做这样的总结：

1. **在当前线程是第一个加入同步队列时，调用compareAndSetHead(new Node())方法，完成链式队列的头结点的初始化**；
2. **自旋不断尝试CAS尾插入节点直至成功为止**。

现在我们已经很清楚获取独占式锁失败的线程包装成Node然后插入同步队列的过程了；那么紧接着会有下一个问题？在同步队列中的节点（线程）会做什么事情了来保证自己能够有机会获得独占式锁了？带着这样的问题我们就来看看acquireQueued()方法，从方法名就可以很清楚，这个方法的作用就是排队获取锁的过程，源码如下：

```java
/**
*  方法的作用是排队获取锁的过程
* 1：如果当前结点的前驱结点是头结点，并且当前线程成功的获取到了同步状态，释放头结点，并将当前结点设置为头结点，failed设置为fasle,返回interrput = false ;if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg),则里面的acquireQueued(addWaiter(Node.EXCLUSIVE), arg)的值为fasle；只要判断的条件是为fasle则当前的线程获取到锁

2：如果线程锁失败
*
*/
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            ////未发生中断
            boolean interrupted = false;
             //仍然通过自旋，根据前面的逻辑，此处传入的为新入列的节点
            for (;;) {
		   // 1. 获得当前节点的先驱节点
            //如果node的前一节点为head节点,而head节点为空节点，说明node是等待队列里排在最前面的节点
            //获取成功
                final Node p = node.predecessor();
				// 2. 当前节点能否获取独占式锁					
				// 2.1 如果当前节点的先驱节点是头结点并且成功获取同步状态，即可以获得独占式锁
                if (p == head && tryAcquire(arg)) {
					//队列头指针用指向当前节点
                    setHead(node);
					//释放前驱节点
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                //判断当前节点的线程是否应该被挂起，如果应该被挂起则挂起。等待release唤醒释放
                //问题：为什么要挂起当前线程？因为如果不挂起的话，线程会一直抢占着CPU
			   // 2.2 获取锁失败，线程进入等待状态等待获取独占式锁
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
}

```

程序逻辑通过注释已经标出，整体来看这是一个这又是一个自旋的过程（for (;;)），代码首先获取当前节点的先驱节点，**如果先驱节点是头结点的并且成功获得同步状态的时候（if (p == head && tryAcquire(arg))），当前节点所指向的线程能够获取锁**。反之，获取锁失败进入等待状态。整体示意图为下图：

那么当获取锁失败的时候会调用shouldParkAfterFailedAcquire()方法和parkAndCheckInterrupt()方法，看看他们做了什么事情。shouldParkAfterFailedAcquire()方法源码为：

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //获取到先驱结点的结点状态
    //该状态值为了简便使用，所以使用了数值类型。非负数值意味着该节点不需要被唤醒。所以，大多数代码中不     //需要检查该状态值的确定值,只需要根据正负值来判断即可对于一个正常的Node，他的waitStatus初始化值      时0.对于一个condition队列中的Node，他的初始化值时CONDITION如果想要修改这个值，可以使用AQS提供      CAS进行修改
    int ws = pred.waitStatus;
    // 前驱节点状态如果为SIGNAL（等待唤醒的状态）。则返回true，然后park挂起线程
    // 当前节点的后继节点已经 (或即将)被阻塞（通过park），所以当当前节点释放或则被取消时候 
    //一定要unpark它的后继节点。为了避免竞争，获取方法一定要首先设置node为signal
    if (ws == Node.SIGNAL)
        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    //表明该节点已经被取消，向前循环重新调整链表节点
    //值大于0表示的是从同步队列中取消(CANCELLED =  1)
    if (ws > 0) {
        /*
         * Predecessor was cancelled. Skip over predecessors and
         * indicate retry.
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
         //执行到这里代表节点是0或者PROPAGATE，然后标记他们为SIGNAL，但是
        //还不能park挂起线程。需要重试是否能获取，如果不能则挂起
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

```

shouldParkAfterFailedAcquire()方法主要逻辑是使用`compareAndSetWaitStatus(pred, ws, Node.SIGNAL)`使用CAS将节点状态由INITIAL设置成SIGNAL，表示当前线程阻塞。**当compareAndSetWaitStatus设置失败则说明shouldParkAfterFailedAcquire方法返回false，然后会在acquireQueued()方法中for (;;)死循环中会继续重试，直至compareAndSetWaitStatus设置节点状态位为SIGNAL时shouldParkAfterFailedAcquire返回true时才会执行方法parkAndCheckInterrupt()方法**，该方法的源码为：

```java
private final boolean parkAndCheckInterrupt() {
        //使得该线程阻塞
		LockSupport.park(this);
        return Thread.interrupted();
}
```

该方法的关键是会调用LookSupport.park()方法（关于LookSupport会在以后的文章进行讨论），该方法是用来阻塞当前线程的。因此到这里就应该清楚了，acquireQueued()在自旋过程中主要完成了两件事情：

1. **如果当前节点的前驱节点是头节点，并且能够获得同步状态的话，当前线程能够获得锁该方法执行结束退出**；
2. **获取锁失败的话，先将节点状态设置成SIGNAL，然后调用LookSupport.park方法使得当前线程阻塞**。

### 3.3 公平的lock方法

```java
final void lock() {
        acquire(1);
    }

    public final void acquire(int arg) {
        //tryAcquirfe方法在FairSync中override，acqiureQueued逻辑不变
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

    protected final boolean tryAcquire(int acquires) {
        //得到当前线程
        final Thread current = Thread.currentThread();
        //获取当前的状态
        int c = getState();
        //如果当前状态等于0就是INIT状态
        if (c == 0) {
            //是否有前置结点hasQueuedPredecessors()如果返回值为true,表示的是这个线程有前置结点，有              线程比当前线程更早的获取锁
            if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
            //如果当前等待队列中没有其他线程且锁状态更新成功则获得锁，立即返回
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

```

### 3.4 unlock方法

```java
    public void unlock() {
        sync.release(1);
    }

    public final boolean release(int arg) {
        //释放锁
        if (tryRelease(arg)) {、
            //获取到队列的头结点
            Node h = head;
            //如果队列的头结点不为空，并且头结点的状态不为0
            if (h != null && h.waitStatus != 0){
            //唤醒最长等待时间的节点
                unparkSuccessor(h);
            }
            return true;
        }
        return false;
    }
```
```java
protected final boolean tryRelease(int releases) {
           //获取到当前线程的状态并且减1
            int c = getState() - releases;
    		//如果进来的线程和当前的线程不一致就会抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
    		//当所有的锁都调用了unlock 状态C才会为0
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
           //设置状态值
            setState(c);
            return free;
        }
```
```java
private void  (Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    //如果小于0（等待状态）就归为0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */

	//s指向的是头节点的后继节点
    Node s = node.next;
    //s为空或者是s的状态大于0，从队列中取消
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
		//后继节点不为null时唤醒该线程
        LockSupport.unpark(s.thread);
}
```

