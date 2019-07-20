# List集合之Vector深度解析

[TOC]

现在还是把Lits中重要的知识点回顾一遍，现在这篇主要讲List集合的三个子类：

- ArrayList
- - 底层数据结构是数组。线程不安全
- LinkedList
- - 底层数据结构是链表。线程不安全
- Vector
- - 底层数据结构是数组。线程安全

其中ArrayList和LinkedList都是用的非常多的

在介绍Vector 的时候，人们常说：

```
底层实现与 ArrayList 类似，不过Vector 是线程安全的，而ArrayList 不是。
```

## 一、Vector解析

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180624/82AA0e2mE2.png?imageslim)

如果我们类比ArrayList的结构图的时候会发现他们直接的结构图上并没有什么差别

###1.1、概览

Vector是 一个矢量队列，它的依赖 关系和ArrayList是一致的。因此它具有以下的功能：

1、**Serializable**：支持对象实现序列化，虽然成员变量没有使用 transient 关键字修饰，Vector 还是实现了 writeObject() 方法进行序列化。

2、**Cloneable**：重写了 clone（）方法，通过 Arrays.copyOf（） 拷贝数组。

3、**RandomAccess**：提供了随机访问功能，我们可以通过元素的序号快速获取元素对象。

4、**AbstractList**：继承了AbstractList ，说明它是一个列表，拥有相应的增，删，查，改等功能。

5、**List**：留一个疑问，为什么继承了 AbstractList 还需要 实现List 接口？

> 关于第5个 问题的答案：
>
> 1、在StackOverFlow 中：[传送门](https://link.juejin.im/?target=http%3A%2F%2Fstackoverflow.com%2Fquestions%2F2165204%2Fwhy-does-linkedhashsete-extend-hashsete-and-implement-sete) 得票最高的答案的回答者说他问了当初写这段代码的 Josh Bloch，得知这就是一个写法错误。 
>
> I've asked Josh Bloch, and he informs me that it was a mistake. He used to think, long ago, that there was some value in it, but he since "saw the light". Clearly JDK maintainers haven't considered this to be worth backing out later. 

 

 ### 1.2、Vector的属性

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180624/9IcCmeLDGF.png?imageslim)

```java
 /**
        与 ArrayList 中一致，elementData 是用于存储数据的。
     */
    protected Object[] elementData;

    /**
     * The number of valid components in this {@code Vector} object.
      与ArrayList 中的size 一样，保存数据的个数
     */
    protected int elementCount;

    /**
     * 设置Vector 的增长系数，如果为空，默认每次扩容2倍。
     *
     * @serial
     */
    protected int capacityIncrement;
    
     // 数组最大值
     private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

```

与ArrayList 中的成员变量相比，Vector 少了两个空数组对象： **EMPTY_ELEMENTDATA** 和 **DEFAULTCAPACITY_EMPTY_ELEMENTDATA**

因此，Vector 与 ArrayList 中的第一个不同点就是，**成员变量不一致**。

 ### 1.3、Vector的 构造函数

Vector 提供了四种构造函数：

- **Vector()**：默认构造函数
- **Vector(int initialCapacity)**：capacity是Vector的默认容量大小。当由于增加数据导致容量增加时，每次容量会增加一倍。
- **Vector(int initialCapacity, int capacityIncrement)**：capacity是Vector的默认容量大小，capacityIncrement是每次Vector容量增加时的增量值。
- **Vector(Collection<? extends E> c)**：创建一个包含collection的Vector

乍一眼看上去，Vector 中提供的构造函数，与ArrayList 中的一样丰富。但是分析过 ArrayList 的构造函数后，再来看Vector 的构造函数，会觉得有一种索然无味的感觉。

 ```java
 //默认构造函数
    public Vector() {
        this(10);
    }
    
    //带初始容量构造函数
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
    
    //带初始容量和增长系数的构造函数
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }


 ```

JDK 1.2 之后提出了将Collection 转换成 Vector 的构造函数，实际操作就是通过Arrays.copyOf() 拷贝一份Collection 数组中的内容到Vector 对象中。这里会有可能抛出 **NullPointerException**。

 

 ### 1.4、Vector的 方法

#### 1.4.1、Vector添加方法

Vector 在添加元素的方法上面，比ArrayList 中多了一个方法。Vector 支持的add 方法有：

- add(E)
- addElement(E)
- add(int i , E element)
- addAll(Collection<? extends E> c)
- addAll(int index, Collection<? extends E> c)

 **4.1 addElement(E)**

我们看一下这个多出来的 **addElement(E)** 方法 有什么特殊之处： 、

```java
 /**
     * Adds the specified component to the end of this vector,
     * increasing its size by one. The capacity of this vector is
     * increased if its size becomes greater than its capacity.
     *
     * <p>This method is identical in functionality to the
     * {@link #add(Object) add(E)}
     * method (which is part of the {@link List} interface).
     *
     * @param   obj   the component to be added
     */
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }

```

从注释上面来看，这个方法就是 跟 add（E） 方法是有着一样的功能的。因此除了返回数据不同外，也没什么特殊之处了。

我们顺着上述代码来进行分析 Vector 中的添加方法。可以看到 Vector 对整个add 方法都上锁了（添加了 **synchronized** 修饰），其实我们可以理解，在添加元素的过程主要包括以下几个操作：

- ensureCapacityHelper（）：确认容器大小
- grow（）：如果有需要，进行容器扩展
- elementData[elementCount++] = obj：设值

为了避免多线程情况下，在 ensureCapacityHelper 容量不需要拓展的情况下，其他线程刚好将数组填满了，这时候就会出现 **ArrayIndexOutOfBoundsException** ，因此对整个方法上锁，就可以避免这种情况出现。

与ArrayList 中对比，确认容器大小这一步骤中，少了 **ArrayList#ensureCapacityInternal** 这一步骤，主要也是源于 Vector 在构造时，以及创建好默认数组大小，不会出现数组为空的情况。

```java
// 确认“Vector容量”的帮助函数
      private void ensureCapacityHelper(int minCapacity) {
         int oldCapacity = elementData.length;
         // 当Vector的容量不足以容纳当前的全部元素，增加容量大小。
         // 若 容量增量系数>0(即capacityIncrement>0)，则将容量增大capacityIncrement
         // 否则，将容量增大一倍。
         if (minCapacity > oldCapacity) {
             Object[] oldData = elementData;
             int newCapacity = (capacityIncrement > 0) ?
                 (oldCapacity + capacityIncrement) : (oldCapacity * 2);
             if (newCapacity < minCapacity) {
                 newCapacity = minCapacity;
             }
             elementData = Arrays.copyOf(elementData, newCapacity);
         }
     }
```

其他的方法也不需要说明了，和ArrayList非常的相似，想研究的话，自己查看源码

 ### 1.5、Vector线程安全问题的研究 

可以说这是与ArrayList的最大的区别所在的地方吧，下面我们研究一下Vector到底是不是线程安全的呢？在StackOverFlow 中有这样一个问题： 

>Is there any danger, if im using one Vector(java.util.Vector) on my server program when im accessing it from multiple threads only for reading? (myvector .size() .get() ...) For writing im using synchronized methods. Thank you. 
>
>其中 一个比较好的回答 是这样回答的  ：
>
>8 down vote
>
>As stated above, every single method of `Vector` is thread-safe by its own because of `synchronized` modifiers. But, if you need some complex operations, such as `get()` or `add()` based on condition which is related to the same `vector`, this is not thread-safe. See example below:
>
>```
>if (vector.size() > 0) {
>    System.out.println(vector.get(0));
>}
>```
>
>This code has a race condition between `size()` and `get()` - the size of vector might be changed by other thread after our thread verified the vector is not empty, and thus `get()` call may return unexpected results. To avoid this, the sample above should be changed like this:
>
>```
>synchronized (vector) {
>    if (vector.size() > 0) {
>        System.out.println(vector.get(0));
>    }
>}
>```
>
>Now this "get-if-not-empty" operation is atomic and race condition-free.
>
>Vector中每一个独立的方法是线程安全的，因为它有着 synchronized的修饰。但是 如果遇到一些比较复杂的操作，比如说是get()或者是 add()方法，并且多个线程需要依靠vector进行相关的判断，那么 这种时候就不是线性安全的。
>
>​      如上述代码所示，Vector 判断完 size（）>0 之后，另一线程如果同时清空vector 对象，那么这时候就会出现异常。因此，在复合操作的情况下，Vector 并不是线程安全的。                                                                            

如果我们没有详细去了解过Vector，或者在面试中，常常会认为Vector 是线程安全的。**但是实际上 Vector 只是在每一个单一方法操作上是线程安全的。** 对于这一句话的理解 我们看下面的一段代码

```java
private static Vector<Integer> vector=new Vector();

    public static void main(String[] args) {
        while(true){
            for(int i=0;i<10;i++){
                vector.add(i); //往vector中添加元素
            }
            Thread removeThread=new Thread(new Runnable() {         
                @Override
                public void run() {
                    //获取vector的大小
                    for(int i=0;i<vector.size();i++){
                        //当前线程让出CPU,使例子中的错误更快出现
                        Thread.yield();
                        //移除第i个数据
                        vector.remove(i);
                    }
                }
            });
            Thread printThread=new Thread(new Runnable() {          
                @Override
                public void run() {
                    //获取vector的大小
                    for(int i=0;i<vector.size();i++){
                        //当前线程让出CPU,使例子中的错误更快出现
                        Thread.yield();
                        //获取第i个数据并打印
                        System.out.println(vector.get(i));
                    }
                }
            });         
            removeThread.start();
            printThread.start();
            //避免同时产生过多线程
            while(Thread.activeCount()>20);
        }
    }
```

运行的结果：、

```java
4
6
6
1
8
3
5
Exception in thread "Thread-285" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 11
    at java.util.Vector.get(Unknown Source)
    at VectorTest$2.run(VectorTest.java:31)
    at java.lang.Thread.run(Unknown Source)
7
9
0
6
```

那么为什么例子中会出现问题呢？这是因为 **例子中有些线程连续调用了两个或两个以上的同步方法** 

假设此时vector大小为5，此时printThread线程执行到 i=4 ，进入for循环但在 `System.out.println(vector.get(i));之前` printThread线程的CPU时间片已到，线程printThread转入就绪状态； 
此时removeThread线程获得CPU开始执行，把vector的5个元素全删除了，这是removeThreadCPU时间片已到； 
接着printThread获得CPU进行执行，由于之前printThread中的i==4，于是调用vector.get(4)获取元素，此时由于vector中的元素已被removeThread线程全部删除，因此报错。

<font color='red'>总的来说，vector保证了其同步方法不能被两个及两个以上线程同时访问</font>

 ## 总结

如果没有深入的研究过vector的源码,只靠那么简单的一句话‘vector是线程安全的’，这样的理解是完全对于 vector的理解是不负责任的，我们看问题更是应该抽丝剥茧

## 参考文献

>[Java 集合系列2、百密一疏之Vector](https://juejin.im/post/5aedcc80518825673c61ccd6)
>
>[Java Vector线程安全？](https://blog.csdn.net/zlp1992/article/details/50433778)



 

 

  

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 