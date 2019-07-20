# 多线程之java的CAS详解

[TOC]

## 前沿

在JDK1.5之前实现同步是靠Synchronized实现的，这会导致锁机制存在以下的问题：

（1）：在多线程的环境下，加锁和释放锁会导致比较多的上下文切换和调度延时，引起性能问题

（2）：一个线程持有锁会导致其它有需要的锁挂起

（3）：如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险

voltaile是不错的机制，但是voltailt只能保证线性的内存的可见性和防止指令重排序，但是并不能保证原子性。因此如果需要同步最终还是需要回到锁的机制上来

**独占锁是一种悲观锁，synchronized就是一种独占锁，会导致其它需要锁的线程挂起，等到持有锁的线程释放锁。**

**乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止，乐观锁用到的机制就是CAS,compare and swap**

## 乐观锁

乐观锁（ `Optimistic Locking`）其实是一种思想。相对悲观锁而言，乐观锁假设认为数据一般情况下不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则让返回用户错误的信息，让用户决定如何去做。 

上面提到的乐观锁的概念中其实已经阐述了他的具体实现细节：主要就是两个步骤：冲突检测和数据更新。其实现方式有一种比较典型的就是Compare and Swap(`CAS`)。 

## 什么是CAS

CAS,compare and swap的缩写，中文翻译成比较并交换 

在Java发展初期，java语言是不能够利用硬件提供的这些便利来提升系统的性能的。而随着java不断的发展**,Java本地方法(JNI)的出现，使得java程序越过JVM直接调用本地方法提供了一种便捷的方式**，因而java在并发的手段上也多了起来。而在Doug Lea提供的cucurenct包中，CAS理论是它实现整个java包的基石。

**CAS是项乐观锁技术**，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。 

**CAS的思想很简单**：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。 

> 这里再强调一下，乐观锁是一种思想。CAS是这种思想的一种实现方式。

## 问题

一个n++的问题。 

```java
public class Case {
 
    public volatile int n;
 
    public void add() {
        n++;
    }
}
```

通过javap -verbose Case看看add方法的字节码指令 

```java
public void add();
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0   //将第一个引用类型本地变量推送至栈顶     
         1: dup       //         
         2: getfield      #2                  // Field n:I
         5: iconst_1      
         6: iadd          
         7: putfield      #2                  // Field n:I
        10: return
```

n++被拆分成了几个指令：

1. 执行getfield拿到原始n；
2. 执行iadd进行加1操作；
3. 执行putfield写把累加后的值写回n；

通过volatile修饰的变量可以保证线程之间的可见性，但并不能保证这3个指令的原子执行，在多线程并发执行下，无法做到线程安全，得到正确的结果，那么应该如何解决呢？ 

## 如何解决

在add方法加上synchronized修饰解决。 

```java
public class Case {
 
    public volatile int n;
 
    public synchronized void add() {
        n++;
    }
}
```

这个方案当然可行，但是性能上差了点，还有其它方案么？ 

```java
public int a = 1;
public boolean compareAndSwapInt(int b) {
    if (a == 1) {
        a = b;
        return true;
    }
    return false;
}
```

如果这段代码在并发下执行，会发生什么？

假设线程1和线程2都过了a==1的检测，都准备执行对a进行赋值，结果就是两个线程同时修改了变量a，显然这种结果是无法符合预期的，无法确定a的最终值。

解决方法也同样暴力，在compareAndSwapInt方法加锁同步，变成一个原子操作，同一时刻只有一个线程才能修改变量a。 

除了低性能的加锁方案，我们还可以使用JDK自带的CAS方案，在CAS中，比较和替换是一组原子操作，不会被外部打断，且在性能上更占有优势 

下面以AtomicInteger的实现为例，分析一下CAS是如何实现的。 

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
 
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
 
    private volatile int value;
    public final int get() {return value;}
}
```

1. Unsafe，是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。
2. 变量valueOffset，表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的。
3. 变量value用volatile修饰，保证了多线程之间的内存可见性。

看看AtomicInteger如何实现并发下的累加操作： 

```java
public final int getAndAdd(int delta) {    
    return unsafe.getAndAddInt(this, valueOffset, delta);
}
 
//unsafe.getAndAddInt
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;

```

假设线程A和线程B同时执行getAndAdd操作（分别跑在不同CPU上）： 

1. AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程A和线程B各自持有一份value的副本，值为3。
2. 线程A通过getIntVolatile(var1, var2)拿到value值3，这时线程A被挂起。
3. 线程B也通过getIntVolatile(var1, var2)方法获取到value值3，运气好，线程B没有被挂起，并执行compareAndSwapInt方法比较内存值也为3，成功修改内存值为2。
4. 这时线程A恢复，执行compareAndSwapInt方法比较，发现自己手里的值(3)和内存的值(2)不一致，说明该值已经被其它线程提前修改过了，那只能重新来一遍了。
5. 重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行compareAndSwapInt进行比较替换，直到成功。

整个过程中，利用CAS保证了对于value的修改的并发安全，继续深入看看Unsafe类中的compareAndSwapInt方法实现。 

```java
public final native boolean compareAndSwapInt(Object paramObject, long paramLong, int paramInt1, int paramInt2);
```

Unsafe类中的compareAndSwapInt，是一个本地方法，该方法的实现位于unsafe.cpp中 

```c++
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```

1. 先想办法拿到变量value在内存中的地址。
2. 通过Atomic::cmpxchg实现比较替换，其中参数x是即将更新的值，参数e是原内存的值。

## Java对CAS的支持

在JDK1.5 中新增`java.util.concurrent`(J.U.C)就是建立在CAS之上的。相对于对于`synchronized`这种阻塞算法，CAS是非阻塞算法的一种常见实现。所以J.U.C在性能上有了很大的提升。 

我们以`java.util.concurrent`中的`AtomicInteger`为例，看一下在不使用锁的情况下是如何保证线程安全的。主要理解`getAndIncrement`方法，该方法的作用相当于 `++i` 操作。 

```java
public class AtomicInteger extends Number implements java.io.Serializable {  
 
    private volatile int value;  
 
    public final int get() {  
        return value;  
    }  
 
    //getAndIncrement采用了CAS操作，
    public final int getAndIncrement() {  
        for (;;) {  
            //每次从内存中读取数据
            int current = get();  
            //然后将此数据和+1后的结果进行CAS操作
            int next = current + 1;  
            //如果成功就返回结果，否则重试直到成功为止
            if (compareAndSet(current, next))  
                return current;  
        }  
    }  
 
    //而compareAndSet利用JNI来完成CPU指令的操作。
    public final boolean compareAndSet(int expect, int update) {  
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  
    }  
}
```



### ABA问题

CAS会导致“ABA问题”。

CAS算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。

问题：如果变量V初次读取的时候是A，并且在准备赋值的时候检查到它仍然是A，那能说明它的值没有被其他线程修改过了吗？

如果在这段期间曾经被改成B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。针对这种情况，java并发包中提供了一个带有标记的原子引用类AtomicStampedReference，它可以通过控制变量值的版本来保证CAS的正确性。