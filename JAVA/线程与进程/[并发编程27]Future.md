# Future

[TOC]



##概述

通过前面几篇的学习，我们知道创建线程的方式有两种，一种是实现Runnable接口，另一种是继承Thread，但是这两种方式都有个缺点，那就是在任务执行完成之后无法获取返回结果，那如果我们想要获取返回结果该如何实现呢？还记上一篇Executor框架结构中提到的Callable接口和Future接口吗？，是的，从JAVA SE 5.0开始引入了Callable和Future，通过它们构建的线程，在任务执行完成后就可以获取执行结果，今天我们就来聊聊线程创建的第三种方式，那就是实现Callable接口。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180809/5Ghj6chIfA.png?imageslim)

###**1.Callable<V>接口**

我们先回顾一下java.lang.Runnable接口，就声明了run(),其返回值为void，当然就无法获取结果了。

```java
public interface Runnable {
    public abstract void run();
}
```

而Callable的接口定义如下 

```java
public interface Callable<V> { 
      V   call()   throws Exception; 
} 
```

该接口声明了一个名称为call()的方法，同时这个方法可以有返回值V，也可以抛出异常。嗯，对该接口我们先了解这么多就行，下面我们来说明如何使用，前篇文章我们说过，无论是Runnable接口的实现类还是Callable接口的实现类，都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行，ThreadPoolExecutor或ScheduledThreadPoolExecutor都实现了ExcutorService接口，而因此Callable需要和Executor框架中的ExcutorService结合使用，我们先看看ExecutorService提供的方法： 

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

第一个方法：submit提交一个实现Callable接口的任务，并且返回封装了异步计算结果的Future。

第二个方法：submit提交一个实现Runnable接口的任务，并且指定了在调用Future的get方法时返回的result对象。

第三个方法：submit提交一个实现Runnable接口的任务，并且返回封装了异步计算结果的Future。

因此我们只要创建好我们的线程对象（实现Callable接口或者Runnable接口），然后通过上面3个方法提交给线程池去执行即可。还有点要注意的是，除了我们自己实现Callable对象外，我们还可以使用工厂类Executors来把一个Runnable对象包装成Callable对象。Executors工厂类提供的方法如下：

```java
public static Callable<Object> callable(Runnable task)
public static <T> Callable<T> callable(Runnable task, T result)
```

####**Callable和Runnable的区别：**

1：Runnable的run方法是被线程调用的，run方法是异步执行的；Callable的call方法不是异步执行的，是由Future的run方法调用的

2：Callable的任务执行后可返回值，而Runnable的任务是不能返回值得。 

3：call方法可以抛出异常，run方法不可以 

4：运行Callable任务可以拿到一个Future对象，Future 表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。计算完成后只能使用 get 方法来获取结果，如果线程没有执行完，Future.get()方法可能会阻塞当前线程的执行；如果线程出现异常，Future.get()会throws InterruptedException或者ExecutionException；如果线程已经取消，会跑出CancellationException。取消由cancel 方法来执行。isDone确定任务是正常完成还是被取消了。一旦计算完成，就不能再取消计算。如果为了可取消性而使用 Future 但又不提供可用的结果，则可以声明Future<?> 形式类型、并返回 null 作为底层任务的结果 



###2.Future<V>接口

Future<V>接口是用来获取异步计算结果的，说白了就是对具体的Runnable或者Callable对象任务执行的结果进行获取(get()),取消(cancel()),判断是否完成等操作。我们看看Future接口的源码：

```java

public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

方法解析：

**V get()** ：获取异步执行的结果，如果没有结果可用，**此方法会阻塞直到异步计算完成**。

**V get(Long timeout , TimeUnit unit)** ：获取异步执行结果，如果没有结果可用，此方法会阻塞，但是会有时间限制，如果阻塞时间超过设定的timeout时间，该方法将抛出异常。

**boolean isDone() ：**如果任务执行结束，无论是正常结束或是中途取消还是发生异常，都返回true。

**boolean isCanceller()** ：如果任务完成前被取消，则返回true。

**boolean cancel(boolean mayInterruptRunning)** ：

**如果任务还没开始**，执行cancel(...)方法将返回false；

**如果任务已经启动**，执行cancel(true)方法将以中断执行此任务线程的方式来试图停止任务，如果停止成功，返回true；

**当任务已经启动**，执行cancel(false)方法将不会对正在执行的任务线程产生影响(让线程正常执行到完成)，此时返回false；

**当任务已经完成**，执行cancel(...)方法将返回false。mayInterruptRunning参数表示是否中断执行中的线程。

通过方法分析我们也知道实际上Future提供了3种功能：

（1）能够中断执行中的任务

（2）判断任务是否执行完成

（3）获取任务执行完成后额结果。

但是我们必须明白Future只是一个接口，我们无法直接创建对象，因此就需要其实现类FutureTask登场啦。

###3.FutureTask类

我们先来看看FutureTask的实现

```java
public class FutureTask<V> implements RunnableFuture<V> {
```

FutureTask类实现了RunnableFuture接口，我们看一下RunnableFuture接口的实现： 

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

分析：FutureTask除了实现了Future接口外还实现了Runnable接口，因此FutureTask也可以直接提交给Executor执行。 当然也可以调用线程直接执行（FutureTask.run()）。接下来我们根据FutureTask.run()的执行时机来分析其所处的3种状态： 

(1）未启动，FutureTask.run()方法还没有被执行之前，FutureTask处于未启动状态，当创建一个FutureTask，而且没有执行FutureTask.run()方法前，这个FutureTask也处于未启动状态。

（2）已启动，FutureTask.run()被执行的过程中，FutureTask处于已启动状态。

（3）已完成，FutureTask.run()方法执行完正常结束，或者被取消或者抛出异常而结束，FutureTask都处于完成状态。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180809/7GC720cBJ7.png?imageslim)

下面我们再来看看FutureTask的方法执行示意图（方法和Future接口基本是一样的，这里就不过多描述了 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180809/b51BFbkf5C.png?imageslim)

（1）当FutureTask处于未启动或已启动状态时，如果此时我们执行FutureTask.get()方法将导致调用线程阻塞；当FutureTask处于已完成状态时，执行FutureTask.get()方法将导致调用线程立即返回结果或者抛出异常。

（2）当FutureTask处于未启动状态时，执行FutureTask.cancel()方法将导致此任务永远不会执行。

当FutureTask处于已启动状态时，执行cancel(true)方法将以中断执行此任务线程的方式来试图停止任务，如果任务取消成功，cancel(...)返回true；但如果执行cancel(false)方法将不会对正在执行的任务线程产生影响(让线程正常执行到完成)，此时cancel(...)返回false。当任务已经完成，执行cancel(...)方法将返回false。

最后我们给出FutureTask的两种构造函数：

```java

public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}
```

### 应用场景

通过上面的介绍，我们对Callable，Future，FutureTask都有了比较清晰的了解了，那么它们到底有什么用呢？我们前面说过通过这样的方式去创建线程的话，最大的好处就是能够返回结果，加入有这样的场景，我们现在需要计算一个数据，而这个数据的计算比较耗时，而我们后面的程序也要用到这个数据结果，那么这个时Callable岂不是最好的选择？我们可以开设一个线程去执行计算，而主线程继续做其他事，而后面需要使用到这个数据时，我们再使用Future获取不就可以了吗？下面我们就来编写一个这样的实例 

#### 实现方式1：

采用线程池的方式，我们可以使用线程池的方式来将当前实现Callable任务通过ThreadPoolExecutor的submit方法添加到线程池，submit方法会返回一个FutureTask对象，而FutureTask是实现了Future接口的，我们可以使用submit返回的FutureTask对象的get方法获取到任务的返回值，其实等会在源码分析的过程中你会发现线程池实现方式只是对我们自己采用线程方式实现的一种封装而已，没什么特别的啦； 

```java

public class CallableDemo implements Callable<Integer> {
	
	private int sum;
	@Override
	public Integer call() throws Exception {
		System.out.println("Callable子线程开始计算啦！");
		Thread.sleep(2000);
		
		for(int i=0 ;i<5000;i++){
			sum=sum+i;
		}
		System.out.println("Callable子线程计算结束！");
		return sum;
	}
}
```

测试代码：

```java
public class CallableTest {
	
	public static void main(String[] args) {
		//创建线程池
		ExecutorService es = Executors.newSingleThreadExecutor();
		//创建Callable对象任务
		CallableDemo calTask=new CallableDemo();
		//提交任务并获取执行结果
		Future<Integer> future =es.submit(calTask);
		//关闭线程池
		es.shutdown();
		try {
			Thread.sleep(2000);
		System.out.println("主线程在执行其他任务");
		
		if(future.get()!=null){
			//输出获取到的结果
			System.out.println("future.get()-->"+future.get());
		}else{
			//输出获取到的结果
			System.out.println("future.get()未获取到结果");
		}
		
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("主线程在执行完成");
	}
}
```

执行的结果是：

```xml
Callable子线程开始计算啦！
主线程在执行其他任务
Callable子线程计算结束！
future.get()-->12497500
主线程在执行完成
```

#### 实现方式二：

 通过单线程的方式我们自己实现，首先创建一个FutureTask对象，在创建FutureTask对象的时候，需要传入一个实现了Callable接口的对象，接着以该FutureTask作为Thread的参数，调用Thread的start方法开启线程执行任务，最后使用FutureTask的get方法获取到任务的返回值就可以了；那么为什么FutureTask对象可以作为Thread的参数呢？原因就在于FutureTask类实现了RunnableFuture接口，而RunnableFuture接口实现了Runnable和Future接口，那么当然可以作为Thread的参数了，我们来看看这种实现方式； 

```java
public class Demo {

    public static void main(String[] args) throws Exception {

        Callable<Integer> call = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                System.out.println("正在计算结果");
                Thread.sleep(3000);
                return 1;
            }
        };

        FutureTask<Integer> task = new FutureTask<>(call);

        Thread thread = new Thread(task);
        
        thread.start();
        
        //do something

        System.out.println("干点别的事情");

        Integer result = task.get();

        System.out.println("拿到的结果为："+result);
    }
}

```



## Future的源码

我们以自己定义线程使用Callable为例，在调用了线程的start方法之后会使得该线程处于就绪状态，有了竞争CPU时间片的权限，当分配到时间片之后，就会执行创建他的参数的run方法，也就是FutureTask的run方法，查看FutureTask的run方法如下`FutureTask<Integer> task = new FutureTask<>(call);`： 

下面的就是Start调用run()方法的过程：

```java

public void run() {
        if (state != NEW ||
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    //call方法的执行
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
```

第12行执行了c的call方法，而c就是我们在创建FutureTask对象的时候传递进来的实现了Callable接口的对象，也就是执行了我们任务的逻辑操作了，将返回的结果赋值给了result，并且在不发生异常的情况下会执行第22行的set方法，来看set方法： 

```java
protected void set(V v) {
    //NEW转态换成ＣＯＭＰＬＴＩＮＧ状态
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            //outcome是object类型的
            outcome = v;
            //状态设置为normal
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
            finishCompletion();
        }
    }
```

set方法做的工作比较简单，就是将结果赋给了outcome而已，outcome是Object类型的全局变量，同时将state状态设置为NORMAL，接着会执行finishCompletion，这个方法其实就是用来唤醒我们上面想要获得数据的get方法的；

**我们先来看看get方法里面的实现：** 

```java
public V get() throws InterruptedException, ExecutionException {
        int s = state;
    //查看当前state状态是否小于等于COMPLETING,如果是表示的是还没有处理完毕
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
    //如果是大于等于COMPLETING就会返回结果，这个结果可能是正常的结果，也可能是异常的结果
        return report(s);
    }
```

 可以看到首先是先去查看当前state状态是否小于等于COMPLETING，你查看FutureTask源码的话，会发现有这么几种状态 

```java
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6;
```

state状态大于COMPLETING，先看看report的方法

```java
 private V report(int s) throws ExecutionException {
        Object x = outcome;
        //如果是正常的完成则把outCome返回
        if (s == NORMAL)
            return (V)x;
     	//如果大于等于CANCELLED，则抛出取消异常
        if (s >= CANCELLED)
            throw new CancellationException();
        throw new ExecutionException((Throwable)x);
    }
```

 state状态小于等于COMPLETING就表示我们刚刚的Callable是还没有处理完成的，那么就会调用awaitDone方法 

```java
 private int awaitDone(boolean timed, long nanos)
        throws InterruptedException {
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        WaitNode q = null;
        boolean queued = false;
        for (;;) {
            //如果当前线程终止了就干掉当前的线程
            if (Thread.interrupted()) {
                removeWaiter(q);
                throw new InterruptedException();
            }
 
            int s = state;
            if (s > COMPLETING) {
                if (q != null)
                    q.thread = null;
                return s;
            }
            else if (s == COMPLETING) // cannot time out yet
                Thread.yield();
            else if (q == null)
                q = new WaitNode();
            else if (!queued)
                queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                     q.next = waiters, q);
            else if (timed) {
                nanos = deadline - System.nanoTime();
                if (nanos <= 0L) {
                    removeWaiter(q);
                    return state;
                }
                LockSupport.parkNanos(this, nanos);
            }
            else
                LockSupport.park(this);
        }
    }
```

 awaitDone的业务逻辑还是挺复杂的，我大致来分析下，第7行判断如果当前线程被中断的话，则抛出异常，第12行获取当前任务状态state，第13行判断如果任务状态大于COMPLETING的话，则直接返回state状态，当然从上面源码中的状态列表中可以发现大于COMPLETING的状态有5种，可能是有正常返回值的，也可能是抛出异常的，具体怎么处理等会在回答get方法的时候是会介绍的，第18行如果任务是正在执行的话，则让出一段CPU时间继续运行，接着第20和22行的判断其实就是判断等待结点和等待队列为空的话创建一个出来而已，接着如果我们在调用awaitDone的时候，设置的timed参数是true的话，则会执行26行处的if语句块，会设置线程等待我们设置的时间，等待时间到了会唤醒此线程，我们平常使用的get方法默认timed值是false的，因此会执行到第34行，将当前线程锁起来，你如果仔细点的话会发现第6到34行是个死循环，也就是当34行处的锁定在其他地方(其实就是set方法里面了)解开的话，仍然会继续执行第6行，那么因为此时state状态已经发生了改变，此时执行已经和之前执行流程不同啦，一般来讲的话，会执行到第13行进行判断，最后执行第16行返回状态码就可以了；

​        那么get调用了LockSupport.park将自己锁住了，那么由谁来解锁呢？答案就是在set方法里面了，在刚刚的set方法的最后会执行finishCompletion方法，这个方法其实就是来解锁的啦：

```java
private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
            if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
                for (;;) {
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }
 
        done();
 
        callable = null;        // to reduce footprint
    }
```

可以看到第9行执行了LockSupport.unpark，相当于解锁了get方法中的LockSupport.park操作，这样的话awaitDone方法就会返回了，回答get方法里面： 