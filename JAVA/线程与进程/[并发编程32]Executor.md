# java线程池和Executor

**概述：**

在Java 5之后，并发编程引入了一堆新的启动、调度和管理线程的API。Executor框架便是Java 5中引入的，其内部使用了线程池机制，它在java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。因此，在Java 5之后，通过Executor来启动线程比使用Thread的start方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，**还有关键的一点**：有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用Executor在构造器中。Executor作为灵活且强大的异步执行框架，其支持多种不同类型的任务执行策略，提供了一种标准的方法将任务的提交过程和执行过程解耦开发，基于生产者-消费者模式，其提交任务的线程相当于生产者，执行任务的线程相当于消费者，并用Runnable来表示任务，Executor的实现还提供了对生命周期的支持，以及统计信息收集，应用程序管理机制和性能监视等机制。 

------



[TOC]



## 一、认识Executor

### 1.1、Executor的UML图

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180810/gC35b01ciF.png?imageslim)

下面是详细的UML图的结构：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180810/6fcgbFhAC5.png?imageslim)

先简单说说这个继承结构：

1、Executor 位于最顶层，也是最简单的，就一个 execute(Runnable runnable) 接口方法定义。

2、ExecutorService 也是接口，在 Executor 接口的基础上添加了很多的接口方法，所以**一般来说我们会使用这个接口**。  

3、然后再下来一层是 AbstractExecutorService，从名字我们就知道，这是抽象类，这里实现了非常有用的一些方法供子类直接使用，之后我们再细说。 

4、下面到我们的重点部分 ThreadPoolExecutor 类，这个类提供了关于线程池所需的非常丰富的功能。

另外，我们还涉及到下图中的这些类： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180810/FL844KcLhi.png?imageslim) 

### 1.2、Executors

同在并发包中的 Executors 类，这个是工具类，里面的方法都是静态方法，如以下我们最常用的用于生成 ThreadPoolExecutor 的实例的一些方法： 

Executors类，提供了一系列工厂方法用于创建线程池，返回的线程池都实现了ExecutorService接口。

1、public static ExecutorService newFiexedThreadPool(int Threads) 创建固定数目线程的线程池。

2、public static ExecutorService newCachedThreadPool()：创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）。如果没有可用的线程，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

3、public static ExecutorService newSingleThreadExecutor()：创建一个单线程化的Executor。

4、public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) :创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

| 类型                        | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| newCachedThreadPool()       | 缓存型池子，先查看池中有没有以前建立的线程，如果有，就使用已经创建的。如果没有，就建一个新的线程加入池中 ；缓存型池子通常用于执行一些**生存期很短**的异步型任务  因此在一些面向连接的daemon型SERVER中用得不多。但对于生存期短的异步任务，它是Executor的首选。 缺省     timeout是60s,超过这个IDLE时长，线程实例将被终止及移出池。注意，放入CachedThreadPool的线程不必担心其结束，超过TIMEOUT不活动，其会自动被终止。 |
| newFixedThreadPool(int)     | -newFixedThreadPool与cacheThreadPool差不多，也是能reuse就用，但不能随时建新的线程 -其独特之处:任意时间点，最多只能有固定数目的活动线程存在，**此时如果有新的线程要建立，只能放在另外的队列中等待**，直到当前的线程中某个线程终止直接被移出池子; 和cacheThreadPool不同，FixedThreadPool没有IDLE机制，所以FixedThreadPool多数针对一些很稳定很固定的正规并发线程，多用于服务器 ;从方法的源代码看，cache池和fixed 池调用的是同一个底层 池，只不过参数不同: fixed池线程数固定，并且是0秒IDLE（无IDLE）     cache池线程数支持Integer.MAX_VALUE(显然完全没考虑主机的资源承受能力），60秒IDLE |
| newScheduledThreadPool(int) | 调度型线程池 -这个池子里的线程可以按schedule依次delay执行，或周期执行 |
| SingleThreadExecutor()      | 单例线程，任意时间池中只能有一个线程 -用的是和cache池和fixed池相同的底层池，但线程数目是1-1,0秒IDLE（无IDLE） |

### 1.3、Executor 和 ExecutorService的区别

**Executor是顶级的接口，Exectors是继承了Executor顶级接口，并且提供了非常多的额外的方法**

- Executor 和 ExecutorService 这两个接口主要的区别是：ExecutorService 接口继承了 Executor 接口，是 Executor 的子接口
- Executor 和 ExecutorService 第二个区别是：**Executor 接口定义了 `execute()`方法用来接收一个`Runnable`接口的对象，而 ExecutorService 接口中的 `submit()`方法可以接受`Runnable`和`Callable`接口的对象**。
- Executor 和 ExecutorService 接口第三个区别是 **Executor 中的 `execute()` 方法不返回任何结果，而 ExecutorService 中的 `submit()`方法可以通过一个 Future 对象返回运算结果。**
- Executor 和 ExecutorService 接口第四个区别是除了允许客户端提交一个任务，ExecutorService 还提供用来控制线程池的方法。比如：调用 `shutDown()` 方法终止线程池。
- Executors 类提供工厂方法用来创建不同类型的线程池。比如: `newSingleThreadExecutor()` 创建一个只有一个线程的线程池，`newFixedThreadPool(int numOfThreads)`来创建固定线程数的线程池，`newCachedThreadPool()`可以根据需要创建新的线程，但如果已有线程是空闲的会重用已有线程。

### 1.4、其他的概念需要知道的

先看看FutureTask的UML图：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180810/5jh6dKeejh.png?imageslim)

　　前篇文章我们说过，无论是Runnable接口的实现类还是Callable接口的实现类，都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行，ThreadPoolExecutor或ScheduledThreadPoolExecutor都实现了ExcutorService接口，而因此Callable需要和Executor框架中的ExcutorService结合使用，我们先看看ExecutorService提供的方法：

 ```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
 ```

第一个方法：submit提交一个实现Callable接口的任务，并且返回封装了异步计算结果的Future。

第二个方法：submit提交一个实现Runnable接口的任务，并且指定了在调用Future的get方法时返回的result对象

第三个方法：submit提交一个实现Runnable接口的任务，并且返回封装了异步计算结果的Future

　　因此我们只要创建好我们的线程对象（实现Callable接口或者Runnable接口），然后通过上面3个方法提交给线程池去执行即可。还有点要注意的是，除了我们自己实现Callable对象外，我们还可以使用工厂类Executors来把一个Runnable对象包装成Callable对象。Executors工厂类提供的方法如下：

```java
public static Callable<Object> callable(Runnable task)
public static <T> Callable<T> callable(Runnable task, T result)
```

　　由于线程池支持**获取线程执行的结果**，所以，引入了 Future 接口，RunnableFuture 继承自此接口，然后我们最需要关心的就是它的实现类 FutureTask。到这里，记住这个概念，在线程池的使用过程中，我们是往线程池提交任务（task），使用过线程池的都知道，我们提交的每个任务是实现了 Runnable 接口的，其实就是先将 Runnable 的任务包装成 FutureTask，然后再提交到线程池。这样，读者才能比较容易记住 FutureTask 这个类名：它首先是一个任务（Task），然后具有 Future 接口的语义，即可以在将来（Future）得到执行的结果。

　　当然，线程池中的 BlockingQueue 也是非常重要的概念，**如果线程数达到 corePoolSize，我们的每个任务会提交到等待队列中，等待线程池中的线程来取任务并执行**。这里的 BlockingQueue 通常我们使用其实现类 LinkedBlockingQueue、ArrayBlockingQueue 和 SynchronousQueue，每个实现类都有不同的特征，使用场景之后会慢慢分析。想要详细了解各个 BlockingQueue 的读者，可以参考我的前面的一篇对 BlockingQueue 的各个实现类进行详细分析的文章。

## 二、源码角度分析Executor

```java
/* 
 * @since 1.5
 * @author Doug Lea
 */
public interface Executor {
    void execute(Runnable command);
}
```

我们可以看到 Executor 接口非常简单，就一个 `void execute(Runnable command)` 方法，代表提交一个任务。为了让大家理解 java 线程池的整个设计方案，我会按照 Doug Lea 的设计思路来多说一些相关的东西。

我们经常这样启动一个线程：

```java
new Thread(new Runnable(){
  // do something
}).start();
```

用了线程池 Executor 后就可以像下面这么使用： 

```java
Executor executor = anExecutor;
executor.execute(new RunnableTask1());
executor.execute(new RunnableTask2());
```

如果我们希望线程池同步执行每一个任务，我们可以这么实现这个接口 :

```java
class DirectExecutor implements Executor {
    public void execute(Runnable r) {
        r.run();// 这里不是用的new Thread(r).start()，也就是说没有启动任何一个新的线程。
    }
}
```

我们希望每个任务提交进来后，直接启动一个新的线程来执行这个任务，我们可以这么实现： 

```java
class ThreadPerTaskExecutor implements Executor {
    public void execute(Runnable r) {
        new Thread(r).start();  // 每个任务都用一个新的线程来执行
    }
}
```

##三、源码角度分析ExecutorService

一般我们定义一个线程池的时候，往往都是使用这个接口：

```java
ExecutorService executor = Executors.newFixOrCachedThreadPool(args...);
```

因为这个接口中定义的一系列方法大部分情况下已经可以满足我们的需要了。

那么我们简单初略地来看一下这个接口中都有哪些方法：

```java
public interface ExecutorService extends Executor {
    // 关闭线程池，已提交的任务继续执行，不接受继续提交新任务
    void shutdown();
    // 关闭线程池，尝试停止正在执行的所有任务，不接受继续提交新任务
    // 它和前面的方法相比，加了一个单词“now”，区别在于它会去停止当前正在进行的任务
    List<Runnable> shutdownNow();
    // 线程池是否已关闭
    boolean isShutdown();
    // 如果调用了 shutdown() 或 shutdownNow() 方法后，所有任务结束了，那么返回true
    // 这个方法必须在调用shutdown或shutdownNow方法之后调用才会返回true
    boolean isTerminated();
    // 等待所有任务完成，并设置超时时间
    // 我们这么理解，实际应用中是，先调用 shutdown 或 shutdownNow，
    // 然后再调这个方法等待所有的线程真正地完成，返回值意味着有没有超时
    boolean awaitTermination(long timeout, TimeUnit unit)
            throws InterruptedException;
    // 提交一个 Callable 任务
    <T> Future<T> submit(Callable<T> task);
    // 提交一个 Runnable 任务，第二个参数将会放到 Future 中，作为返回值，
    // 因为 Runnable 的 run 方法本身并不返回任何东西
    <T> Future<T> submit(Runnable task, T result);
    // 提交一个 Runnable 任务
    Future<?> submit(Runnable task);
    // 执行所有任务，返回 Future 类型的一个 list
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
            throws InterruptedException;
    // 也是执行所有任务，但是这里设置了超时时间
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
            throws InterruptedException;
    // 只有其中的一个任务结束了，就可以返回，返回执行完的那个任务的结果
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
            throws InterruptedException, ExecutionException;
  
    // 同上一个方法，只有其中的一个任务结束了，就可以返回，返回执行完的那个任务的结果，
    // 不过这个带超时，超过指定的时间，抛出 TimeoutException 异常
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
            throws InterruptedException, ExecutionException, TimeoutException;
}
```

这里有几点需要说明一下的：

isShutdown()和isTerminated()的区别:

　　isShutdown()是在调用完shutdown()方法返回true；

　　isTerminated()：当调用shutdown（）方法后，并且所有提交的任务完成返回为true

## 四：再说一下FutureTask

在继续往下层介绍 ExecutorService 的实现类之前，我们先来说说相关的类 FutureTask。 

```java
Future   -> RunnableFuture -> FutureTask
Runnable -> RunnableFuture
FutureTask 通过 RunnableFuture 间接实现了 Runnable 接口，
所以每个 Runnable 通常都先包装成 FutureTask，
然后调用 executor.execute(Runnable command) 将其提交给线程池
```

我们知道，Runnable 的 void run() 方法是没有返回值的，所以，通常，如果我们需要的话，会在 submit 中指定第二个参数作为返回值： 

```java
<T> Future<T> submit(Runnable task, T result);
```

其实到时候会通过这两个参数，将其包装成 Callable。

Callable 也是因为线程池的需要，所以才有了这个接口。它和 Runnable 的区别在于 run() 没有返回值，而 Callable 的 call() 方法有返回值，同时，如果运行出现异常，call() 方法会抛出异常。

```java
public interface Callable<V> {
   
    V call() throws Exception;
}
```

具体的Future的分析在我的前面的博客中已经写到了：

## 五：AbstractExecutorService

AbstractExecutorService 抽象类派生自 ExecutorService 接口，然后在其基础上实现了几个实用的方法，这些方法提供给子类进行调用。

这个抽象类实现了 invokeAny 方法和 invokeAll 方法，这里的两个 newTaskFor 方法也比较有用，用于将任务包装成 FutureTask。定义于最上层接口 Executor中的 `void execute(Runnable command)` 由于不需要获取结果，不会进行 FutureTask 的包装。

> 需要获取结果（FutureTask），用 submit 方法，不需要获取结果，可以用 execute 方法。 

下面，我将一行一行源码地来分析这个类，跟着源码来看看其实现吧： 

> Tips: invokeAny 和 invokeAll 方法占了这整个类的绝大多数篇幅，读者可以选择适当跳过，因为它们可能在你的实践中使用的频次比较低，而且它们不带有承前启后的作用，不用担心会漏掉什么导致看不懂后面的代码。  

```java
public abstract class AbstractExecutorService implements ExecutorService {
    // RunnableFuture 是用于获取执行结果的，我们常用它的子类 FutureTask
    // 下面两个 newTaskFor 方法用于将我们的任务包装成 FutureTask 提交到线程池中执行
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
    // 提交任务
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        // 1. 将任务包装成 FutureTask
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        // 2. 交给执行器执行，execute 方法由具体的子类来实现
        // 前面也说了，FutureTask 间接实现了Runnable 接口。
        execute(ftask);
        return ftask;
    }
    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        // 1. 将任务包装成 FutureTask
        RunnableFuture<T> ftask = newTaskFor(task, result);
        // 2. 交给执行器执行
        execute(ftask);
        return ftask;
    }
  
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        // 1. 将任务包装成 FutureTask
        RunnableFuture<T> ftask = newTaskFor(task);
        // 2. 交给执行器执行
        execute(ftask);
        return ftask;
    }
    // 此方法目的：将 tasks 集合中的任务提交到线程池执行，任意一个线程执行完后就可以结束了
    // 第二个参数 timed 代表是否设置超时机制，超时时间为第三个参数，
    // 如果 timed 为 true，同时超时了还没有一个线程返回结果，那么抛出 TimeoutException 异常
    private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks,
                            boolean timed, long nanos)
        throws InterruptedException, ExecutionException, TimeoutException {
        if (tasks == null)
            throw new NullPointerException();
        // 任务数
        int ntasks = tasks.size();
        if (ntasks == 0)
            throw new IllegalArgumentException();
        // 
        List<Future<T>> futures= new ArrayList<Future<T>>(ntasks);
      
        // ExecutorCompletionService 不是一个真正的执行器，参数this才是真正的执行器
        // 它对执行器进行了包装，每个任务结束后，将结果保存到内部的一个 completionQueue 队列中
        // 这也是为什么这个类的名字里面有个 Completion 的原因吧。
        ExecutorCompletionService<T> ecs =
            new ExecutorCompletionService<T>(this);
        try {
            // 用于保存异常信息，此方法如果没有得到任何有效的结果，那么我们可以抛出最后得到的一个异常
            ExecutionException ee = null;
            long lastTime = timed ? System.nanoTime() : 0;
            Iterator<? extends Callable<T>> it = tasks.iterator();
            // 首先先提交一个任务，后面的任务到下面的 for 循环一个个提交
            futures.add(ecs.submit(it.next()));
            // 提交了一个任务，所以任务数量减 1
            --ntasks;
            // 正在执行的任务数(提交的时候 +1，任务结束的时候 -1)
            int active = 1;
            for (;;) {
                // ecs 上面说了，其内部有一个 completionQueue 用于保存执行完成的结果
                // BlockingQueue 的 poll 方法不阻塞，返回 null 代表队列为空
                Future<T> f = ecs.poll();
                // 为 null，说明刚刚提交的第一个线程还没有执行完成
                // 在前面先提交一个任务，加上这里做一次检查，也是为了提高性能
                if (f == null) {
                    if (ntasks > 0) {
                        --ntasks;
                        futures.add(ecs.submit(it.next()));
                        ++active;
                    }
                    // 这里是 else if，不是 if。这里说明，没有任务了，同时 active 为 0 说明
                    // 任务都执行完成了。其实我也没理解为什么这里做一次 break？
                    // 因为我认为 active 为 0 的情况，必然从下面的 f.get() 返回了
                    else if (active == 0)
                        break;
                    // 这里也是 else if。这里说的是，没有任务了，但是设置了超时时间，这里检测是否超时
                    else if (timed) {
                        // 带等待的 poll 方法
                        f = ecs.poll(nanos, TimeUnit.NANOSECONDS);
                        // 如果已经超时，抛出 TimeoutException 异常，这整个方法就结束了
                        if (f == null)
                            throw new TimeoutException();
                        long now = System.nanoTime();
                        nanos -= now - lastTime;
                        lastTime = now;
                    }
                    // 这里是 else。说明，没有任务需要提交，但是池中的任务没有完成，还没有超时(如果设置了超时)
                    // take() 方法会阻塞，直到有元素返回，说明有任务结束了
                    else
                        f = ecs.take();
                }
                /*
                 * 我感觉上面这一段并不是很好理解，这里简单说下。
                 * 1. 首先，这在一个 for 循环中，我们设想每一个任务都没那么快结束，
                 *     那么，每一次都会进到第一个分支，进行提交任务，直到将所有的任务都提交了
                 * 2. 任务都提交完成后，如果设置了超时，那么 for 循环其实进入了“一直检测是否超时”
                       这件事情上
                 * 3. 如果没有设置超时机制，那么不必要检测超时，那就会阻塞在 ecs.take() 方法上，
                       等待获取第一个执行结果
                 * ?. 这里我还没理解 active == 0 这个分支的到底是干嘛的？
                 */
              
              
              
                // 有任务结束了
                if (f != null) {
                    --active;
                    try {
                        // 返回执行结果，如果有异常，都包装成 ExecutionException
                        return f.get();
                    } catch (ExecutionException eex) {
                        ee = eex;
                    } catch (RuntimeException rex) {
                        ee = new ExecutionException(rex);
                    }
                }
            }// 注意看 for 循环的范围，一直到这里
          
            if (ee == null)
                ee = new ExecutionException();
            throw ee;
		
        } finally {
            // 方法退出之前，取消其他的任务
            for (Future<T> f : futures)
                f.cancel(true);
        }
    }
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
        try {
            return doInvokeAny(tasks, false, 0);
        } catch (TimeoutException cannotHappen) {
            assert false;
            return null;
        }
    }
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
        return doInvokeAny(tasks, true, unit.toNanos(timeout));
    }
    // 执行所有的任务，返回任务结果。
    // 先不要看这个方法，我们先想想，其实我们自己提交任务到线程池，也是想要线程池执行所有的任务
    // 只不过，我们是每次 submit 一个任务，这里以一个集合作为参数提交
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        if (tasks == null)
            throw new NullPointerException();
        List<Future<T>> futures = new ArrayList<Future<T>>(tasks.size());
        boolean done = false;
        try {
            // 这个很简单
            for (Callable<T> t : tasks) {
                // 包装成 FutureTask
                RunnableFuture<T> f = newTaskFor(t);
                futures.add(f);
                // 提交任务
                execute(f);
            }
            for (Future<T> f : futures) {
                if (!f.isDone()) {
                    try {
                        // 这是一个阻塞方法，直到获取到值，或抛出了异常
                        // 这里有个小细节，其实 get 方法签名上是会抛出 InterruptedException 的
                        // 可是这里没有进行处理，而是抛给外层去了。此异常发生于还没执行完的任务被取消了
                        f.get();
                    } catch (CancellationException ignore) {
                    } catch (ExecutionException ignore) {
                    }
                }
            }
            done = true;
            // 这个方法返回，不像其他的场景，返回 List<Future>，其实执行结果还没出来
            // 这个方法返回是真正的返回，任务都结束了
            return futures;
        } finally {
            // 为什么要这个？就是上面说的有异常的情况
            if (!done)
                for (Future<T> f : futures)
                    f.cancel(true);
        }
    }
    // 带超时的 invokeAll，我们找不同吧
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
        if (tasks == null || unit == null)
            throw new NullPointerException();
        long nanos = unit.toNanos(timeout);
        List<Future<T>> futures = new ArrayList<Future<T>>(tasks.size());
        boolean done = false;
        try {
            for (Callable<T> t : tasks)
                futures.add(newTaskFor(t));
            long lastTime = System.nanoTime();
            Iterator<Future<T>> it = futures.iterator();
            // 提交一个任务，检测一次是否超时
            while (it.hasNext()) {
                execute((Runnable)(it.next()));
                long now = System.nanoTime();
                nanos -= now - lastTime;
                lastTime = now;
                // 超时
                if (nanos <= 0)
                    return futures;
            }
            for (Future<T> f : futures) {
                if (!f.isDone()) {
                    if (nanos <= 0)
                        return futures;
                    try {
                        // 调用带超时的 get 方法，这里的参数 nanos 是剩余的时间，
                        // 因为上面其实已经用掉了一些时间了
                        f.get(nanos, TimeUnit.NANOSECONDS);
                    } catch (CancellationException ignore) {
                    } catch (ExecutionException ignore) {
                    } catch (TimeoutException toe) {
                        return futures;
                    }
                    long now = System.nanoTime();
                    nanos -= now - lastTime;
                    lastTime = now;
                }
            }
            done = true;
            return futures;
        } finally {
            if (!done)
                for (Future<T> f : futures)
                    f.cancel(true);
        }
    }
}
```

一下是上面的代码的简单的分析，如果想要非常详细的了解具体的实现的细节的东西，可以阅读源码：

1：从4-9行代码，两个newTaskFor方法用于将我们的任务包装成FutureTask提交到线程池中执行

2：从11-36行代码，提交任务的代码，大致的步骤就是分为两步，第一步：将任务包装成FutureTask  第二步：将给执行器去执行，方法由具体的子类实现；

3：从40-137行代码，doInvokeAny（）将tasks集合中的任务提交到线程池中执行，任意一个线程执行完后就可以结束了

　　到这里，我们发现，这个抽象类包装了一些基本的方法，可是像 submit、invokeAny、invokeAll 等方法，它们都没有真正开启线程来执行任务，它们都只是在方法内部调用了 execute 方法，所以最重要的 execute(Runnable runnable) 方法还没出现，需要等具体执行器来实现这个最重要的部分，这里我们要说的就是 ThreadPoolExecutor 类了。 

## 六、ThreadPoolExecutor

ThreadPoolExecutor 是 JDK 中的线程池实现，这个类实现了一个线程池需要的各个方法，它实现了任务提交、线程管理、监控等等方法。

我们可以基于它来进行业务上的扩展，以实现我们需要的其他功能，比如实现定时任务的类 ScheduledThreadPoolExecutor 就继承自 ThreadPoolExecutor。当然，这不是本文关注的重点，下面，还是赶紧进行源码分析吧。

首先，我们来看看线程池实现中的几个概念和处理流程。

我们先回顾下提交任务的几个方法：

```java
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

一个最基本的概念是，submit 方法中，参数是 Runnable 类型（也有Callable 类型），这个参数不是用于 new Thread(**runnable**).start() 中的，此处的这个参数不是用于启动线程的，这里指的是**任务**，任务要做的事情是 run() 方法里面定义的或 Callable 中的 call() 方法里面定义的。

初学者往往会搞混这个，因为 Runnable 总是在各个地方出现，经常把一个 Runnable 包到另一个 Runnable 中。请把它想象成有个 Task 接口，这个接口里面有一个 run() 方法（我想作者只是不想因为这个再定义一个完全可以用 Runnable 来代替的接口，Callable 的出现，完全是因为 Runnable 不能满足需要）。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180810/FeG0Fbgfgi.png?imageslim)

当然，上图没有考虑队列是否有界，提交任务时队列满了怎么办？什么情况下会创建新的线程？提交任务时线程池满了怎么办？空闲线程怎么关掉？这些问题下面我们会一一解决。 

先看看构造方法：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    // 这几个参数都是必须要有的
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
  
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

基本上，上面的构造方法中列出了我们最需要关心的几个属性了，下面逐个介绍下构造方法中出现的这几个属性：

- corePoolSize

  > 核心线程数，

- maximumPoolSize

  > 最大线程数，线程池允许创建的最大线程数。

- workQueue

  > 任务队列，BlockingQueue 接口的某个实现（常使用 ArrayBlockingQueue 和 LinkedBlockingQueue）。

- keepAliveTime

  > 空闲线程的保活时间，如果某线程的空闲时间超过这个值都没有任务给它做，那么可以被关闭了。注意这个值并不会对所有线程起作用，如果线程池中的线程数少于等于核心线程数 corePoolSize，那么这些线程不会因为空闲太长时间而被关闭，当然，也可以通过调用 `allowCoreThreadTimeOut(true)`使核心线程数内的线程也可以被回收。

- threadFactory

  > 用于生成线程，一般我们可以用默认的就可以了。通常，我们可以通过它将我们的线程的名字设置得比较可读一些，如 Message-Thread-1， Message-Thread-2 类似这样。

- handler：

  > 当线程池已经满了，但是又有新的任务提交的时候，该采取什么策略由这个来指定。有几种方式可供选择，像抛出异常、直接拒绝然后返回等，也可以自己实现相应的接口实现自己的逻辑，这个之后再说。
  >
  > RejectedExecutionHandler（**饱和策略**）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。
  >
  > - CallerRunsPolicy：只用调用者所在线程来运行任务。
  > - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
  > - DiscardPolicy：不处理，丢弃掉。
  > - 当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。

除了上面几个属性外，我们再看看其他重要的属性。

Doug Lea 采用一个 32 位的整数来存放线程池的状态和当前池中的线程数，其中高 3 位用于存放线程池状态，低 29 位表示线程数（即使只有 29 位，也已经不小了，大概 500 多万）。我们知道，java 语言在整数编码上是统一的，都是采用补码的形式，下面是简单的移位操作和布尔操作，都是挺简单的。 

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
// 这里 COUNT_BITS 设置为 29(32-3)，意味着前三位用于存放线程状态，后29位用于存放线程数
// 很多初学者很喜欢在自己的代码中写很多 29 这种数字，或者某个特殊的字符串，然后分布在各个地方，这是非常糟糕的
private static final int COUNT_BITS = Integer.SIZE - 3;
// 000 11111111111111111111111111111
// 这里得到的是 29 个 1，也就是说线程池的最大线程数是 2^29-1=536860911
// 以我们现在计算机的实际情况，这个数量还是够用的
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
// 我们说了，线程池的状态存放在高 3 位中
// 运算结果为 111跟29个0：111 00000000000000000000000000000
private static final int RUNNING    = -1 << COUNT_BITS;
// 000 00000000000000000000000000000
private static final int SHUTDOWN   =  0 << COUNT_BITS;
// 001 00000000000000000000000000000
private static final int STOP       =  1 << COUNT_BITS;
// 010 00000000000000000000000000000
private static final int TIDYING    =  2 << COUNT_BITS;
// 011 00000000000000000000000000000
private static final int TERMINATED =  3 << COUNT_BITS;
// 将整数 c 的低 29 位修改为 0，就得到了线程池的状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
// 将整数 c 的高 3 为修改为 0，就得到了线程池中的线程数
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
/*
 * Bit field accessors that don't require unpacking ctl.
 * These depend on the bit layout and on workerCount being never negative.
 */
private static boolean runStateLessThan(int c, int s) {
    return c < s;
}
private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
}
private static boolean isRunning(int c) {
    return c < SHUTDOWN;
}
```

上面就是对一个整数的简单的位操作，几个操作方法将会在后面的源码中一直出现，所以读者最好把方法名字和其代表的功能记住，看源码的时候也就不需要来来回回翻了。

在这里，介绍下线程池中的各个状态和状态变化的转换过程：

- RUNNING：这个没什么好说的，这是最正常的状态：接受新的任务，处理等待队列中的任务
- SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务
- STOP：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程
- TIDYING：所有的任务都销毁了，workCount 为 0。线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()
- TERMINATED：terminated() 方法结束后，线程池的状态就会变成这个

> RUNNING 定义为 -1，SHUTDOWN 定义为 0，其他的都比 0 大，所以等于 0 的时候不能提交任务，大于 0 的话，连正在执行的任务也需要中断。 

看了这几种状态的介绍，读者大体也可以猜到十之八九的状态转换了，各个状态的转换过程有以下几种：

- RUNNING -> SHUTDOWN：当调用了 shutdown() 后，会发生这个状态转换，这也是最重要的
- (RUNNING or SHUTDOWN) -> STOP：当调用 shutdownNow() 后，会发生这个状态转换，这下要清楚 shutDown() 和 shutDownNow() 的区别了
- SHUTDOWN -> TIDYING：当任务队列和线程池都清空后，会由 SHUTDOWN 转换为 TIDYING
- STOP -> TIDYING：当任务队列清空后，发生这个转换
- TIDYING -> TERMINATED：这个前面说了，当 terminated() 方法结束后

上面的几个记住核心的就可以了，尤其第一个和第二个。

另外，我们还要看看一个内部类 Worker，因为 Doug Lea 把线程池中的线程包装成了一个个 Worker，翻译成工人，就是线程池中做任务的线程。所以到这里，我们知道**任务是 Runnable（内部叫 task 或 command），线程是 Worker**。 

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable
{
    private static final long serialVersionUID = 6138294804551838833L;
    // 这个是真正的线程，任务靠你啦
    final Thread thread;
    
    // 前面说了，这里的 Runnable 是任务。为什么叫 firstTask？因为在创建线程的时候，如果同时指定了
    // 这个线程起来以后需要执行的第一个任务，那么第一个任务就是存放在这里的(线程可不止执行这一个任务)
    // 当然了，也可以为 null，这样线程起来了，自己到任务队列（BlockingQueue）中取任务（getTask 方法）就行了
    Runnable firstTask;
    
    // 用于存放此线程完成的任务数，注意了，这里用了 volatile，保证可见性
    volatile long completedTasks;
	// Worker 只有这一个构造方法，传入 firstTask，也可以传 null
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        // 调用 ThreadFactory 来创建一个新的线程
        this.thread = getThreadFactory().newThread(this);
    }
    // 这里调用了外部类的 runWorker 方法
    public void run() {
        runWorker(this);
    }
	...// 其他几个方法没什么好看的，就是用 AQS 操作，来获取这个线程的执行权，用了独占锁
}
```

前面虽然啰嗦，但是简单。有了上面的这些基础后，我们终于可以看看 ThreadPoolExecutor 的 execute 方法了，前面源码分析的时候也说了，各种方法都最终依赖于 execute 方法： 

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
  
    // 前面说的那个表示 “线程池状态” 和 “线程数” 的整数
    int c = ctl.get();
  
    // 如果当前线程数少于核心线程数，那么直接添加一个 worker 来执行任务，
    // 创建一个新的线程，并把当前任务 command 作为这个线程的第一个任务(firstTask)
    if (workerCountOf(c) < corePoolSize) {
        // 添加任务成功，那么就结束了。提交任务嘛，线程池已经接受了这个任务，这个方法也就可以返回了
        // 至于执行的结果，到时候会包装到 FutureTask 中。
        // 返回 false 代表线程池不允许提交任务
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 到这里说明，要么当前线程数大于等于核心线程数，要么刚刚 addWorker 失败了
  
    // 如果线程池处于 RUNNING 状态，把这个任务添加到任务队列 workQueue 中
    if (isRunning(c) && workQueue.offer(command)) {
        /* 这里面说的是，如果任务进入了 workQueue，我们是否需要开启新的线程
         * 因为线程数在 [0, corePoolSize) 是无条件开启新的线程
         * 如果线程数已经大于等于 corePoolSize，那么将任务添加到队列中，然后进到这里
         */
        int recheck = ctl.get();
        // 如果线程池已不处于 RUNNING 状态，那么移除已经入队的这个任务，并且执行拒绝策略
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 如果线程池还是 RUNNING 的，并且线程数为 0，那么开启新的线程
        // 到这里，我们知道了，这块代码的真正意图是：担心任务提交到队列中了，但是线程都关闭了
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果 workQueue 队列满了，那么进入到这个分支
    // 以 maximumPoolSize 为界创建新的 worker，
    // 如果失败，说明当前线程数已经达到 maximumPoolSize，执行拒绝策略
    else if (!addWorker(command, false))
        reject(command);
}
```

对创建线程的错误理解：如果线程数少于 corePoolSize，创建一个线程，如果线程数在 [corePoolSize, maximumPoolSize] 之间那么可以创建线程或复用空闲线程，keepAliveTime 对这个区间的线程有效。从上面的几个分支，我们就可以看出，上面的这段话是错误的。

**根据ThreadPoolExecutor源码前面大段的注释，我们可以看出，当试图通过excute方法将一个Runnable任务添加到线程池中时，按照如下顺序来处理：**

​    1、如果线程池中的线程数量少于corePoolSize，即使线程池中有空闲线程，也会创建一个新的线程来执行新添加的任务；

​    2、如果线程池中的线程数量大于等于corePoolSize，但缓冲队列workQueue未满，则将新添加的任务放到workQueue中，按照FIFO的原则依次等待执行（线程池中有线程空闲出来后依次将缓冲队列中的任务交付给空闲的线程执行）；

   3、如果线程池中的线程数量大于等于corePoolSize，且缓冲队列workQueue已满，但线程池中的线程数量小于maximumPoolSize，则会创建新的线程来处理被添加的任务；

  4、如果线程池中的线程数量等于了maximumPoolSize，有4种处理方式（该构造方法调用了含有5个参数的构造方法，并将最后一个构造方法为RejectedExecutionHandler类型，它在处理线程溢出时有4种方式，这里不再细说，要了解的，自己可以阅读下源码）。

​    总结起来，也即是说，当有新的任务要处理时，先看线程池中的线程数量是否大于corePoolSize，再看缓冲队列workQueue是否满，最后看线程池中的线程数量是否大于maximumPoolSize。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180810/Bb076ImkjF.png?imageslim)

这个方法非常重要 addWorker(Runnable firstTask, boolean core) 方法，我们看看它是怎么创建新的线程的： 

```java
// 第一个参数是准备提交给这个线程执行的任务，之前说了，可以为 null
// 第二个参数为 true 代表使用核心线程数 corePoolSize 作为创建线程的界线，也就说创建这个线程的时候，
// 		如果线程池中的线程总数已经达到 corePoolSize，那么不能响应这次创建线程的请求
// 		如果是 false，代表使用最大线程数 maximumPoolSize 作为界线
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        // 这个非常不好理解
        // 如果线程池已关闭，并满足以下条件之一，那么不创建新的 worker：
        // 1. 线程池状态大于 SHUTDOWN，其实也就是 STOP, TIDYING, 或 TERMINATED
        // 2. firstTask != null
        // 3. workQueue.isEmpty()
        // 简单分析下：
        // 还是状态控制的问题，当线程池处于 SHUTDOWN 的时候，不允许提交任务，但是已有的任务继续执行
        // 当状态大于 SHUTDOWN 时，不允许提交任务，且中断正在执行的任务
        // 多说一句：如果线程池处于 SHUTDOWN，但是 firstTask 为 null，且 workQueue 非空，那么是允许创建 worker 的
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;
        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 如果成功，那么就是所有创建线程前的条件校验都满足了，准备创建线程执行任务了
            // 这里失败的话，说明有其他线程也在尝试往线程池中创建线程
            if (compareAndIncrementWorkerCount(c))
                break retry;
            // 由于有并发，重新再读取一下 ctl
            c = ctl.get();
            // 正常如果是 CAS 失败的话，进到下一个里层的for循环就可以了
            // 可是如果是因为其他线程的操作，导致线程池的状态发生了变更，如有其他线程关闭了这个线程池
            // 那么需要回到外层的for循环
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    /* 
     * 到这里，我们认为在当前这个时刻，可以开始创建线程来执行任务了，
     * 因为该校验的都校验了，至于以后会发生什么，那是以后的事，至少当前是满足条件的
     */
  
    // worker 是否已经启动
    boolean workerStarted = false;
    // 是否已将这个 worker 添加到 workers 这个 HashSet 中
    boolean workerAdded = false;
    Worker w = null;
    try {
        final ReentrantLock mainLock = this.mainLock;
        // 把 firstTask 传给 worker 的构造方法
        w = new Worker(firstTask);
        // 取 worker 中的线程对象，之前说了，Worker的构造方法会调用 ThreadFactory 来创建一个新的线程
        final Thread t = w.thread;
        if (t != null) {
            // 这个是整个类的全局锁，持有这个锁才能让下面的操作“顺理成章”，
            // 因为关闭一个线程池需要这个锁，至少我持有锁的期间，线程池不会被关闭
            mainLock.lock();
            try {
                int c = ctl.get();
                int rs = runStateOf(c);
                // 小于 SHUTTDOWN 那就是 RUNNING，这个自不必说，是最正常的情况
                // 如果等于 SHUTDOWN，前面说了，不接受新的任务，但是会继续执行等待队列中的任务
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    // worker 里面的 thread 可不能是已经启动的
                    if (t.isAlive())
                        throw new IllegalThreadStateException();
                    // 加到 workers 这个 HashSet 中
                    workers.add(w);
                    int s = workers.size();
                    // largestPoolSize 用于记录 workers 中的个数的最大值
                    // 因为 workers 是不断增加减少的，通过这个值可以知道线程池的大小曾经达到的最大值
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 添加成功的话，启动这个线程
            if (workerAdded) {
                // 启动线程
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // 如果线程没有启动，需要做一些清理工作，如前面 workCount 加了 1，将其减掉
        if (! workerStarted)
            addWorkerFailed(w);
    }
    // 返回线程是否启动成功
    return workerStarted;
}
```

## 总结

1. java 线程池有哪些关键属性？

   > corePoolSize，maximumPoolSize，workQueue，keepAliveTime，rejectedExecutionHandler
   >
   > corePoolSize 到 maximumPoolSize 之间的线程会被回收，当然 corePoolSize 的线程也可以通过设置而得到回收（allowCoreThreadTimeOut(true)）。
   >
   > workQueue 用于存放任务，添加任务的时候，如果当前线程数超过了 corePoolSize，那么往该队列中插入任务，线程池中的线程会负责到队列中拉取任务。
   >
   > keepAliveTime 用于设置空闲时间，如果线程数超出了 corePoolSize，并且有些线程的空闲时间超过了这个值，会执行关闭这些线程的操作
   >
   > rejectedExecutionHandler 用于处理当线程池不能执行此任务时的情况，默认有**抛出 RejectedExecutionException 异常**、**忽略任务**、**使用提交任务的线程来执行此任务**和**将队列中等待最久的任务删除，然后提交此任务**这四种策略，默认为抛出异常。

2. 说说线程池中的线程创建时机？

   > 1. 如果当前线程数少于 corePoolSize，那么提交任务的时候创建一个新的线程，并由这个线程执行这个任务；
   > 2. 如果当前线程数已经达到 corePoolSize，那么将提交的任务添加到队列中，等待线程池中的线程去队列中取任务；
   > 3. 如果队列已满，那么创建新的线程来执行任务，需要保证池中的线程数不会超过 maximumPoolSize，如果此时线程数超过了 maximumPoolSize，那么执行拒绝策略。

3. Executors.newFixedThreadPool(…) 和 Executors.newCachedThreadPool() 构造出来的线程池有什么差别？

   > 和cacheThreadPool不同，FixedThreadPool没有IDLE机制，所以FixedThreadPool多数针对一些很稳定很固定的正规并发线程

4. 任务执行过程中发生异常怎么处理？

   > 如果某个任务执行出现异常，那么执行任务的线程会被关闭，而不是继续接收其他任务。然后会启动一个新的线程来代替它。

5. 什么时候会执行拒绝策略？

   > 1. workers 的数量达到了 corePoolSize，任务入队成功，以此同时线程池被关闭了，而且关闭线程池并没有将这个任务出队，那么执行拒绝策略。这里说的是非常边界的问题，入队和关闭线程池并发执行，读者仔细看看 execute 方法是怎么进到第一个 reject(command) 里面的。
   > 2. workers 的数量大于等于 corePoolSize，准备入队，可是队列满了，任务入队失败，那么准备开启新的线程，可是线程数已经达到 maximumPoolSize，那么执行拒绝策略。





