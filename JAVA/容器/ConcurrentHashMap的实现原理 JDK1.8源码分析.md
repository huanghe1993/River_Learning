# ConcurrentHashMap的实现原理 JDK1.8源码分析

[TOC]

## 一、首先分析一下并发情况下为什么使用ConcurrentHashMap

1、HashMap在高并发的情况下，执行put操作在进行扩容的时候会导致HashMap的Entry链表形成环形数据结构，从而导致Entry的next结点始终不为空，因此产生死循环

2、HashTable虽然是线程安全的，但是效率低下，当一个线程访问hashTable的同步方法时，其他线程如果也访问HashTable的同步方法，那么会进入到阻塞或者是轮询转态

3、在JDK1.6中ConcurrentHashMap使用锁分段技术提高并发访问效率 。首先将数据分成一段一段的存储，然后给每一段配一个锁，当一个线程占用锁访问其中的一段数据时，其他 段的数据也能被其他的线程访问到。然而在在JDK1.8中的实现以及抛弃了Segment分段锁的机制，利用CAS+Synchronized来保证并发更新的安全，底层仍然采用的是数组+链表+红黑树的数据结构

##二、JDK1.7的ConcurrentHashMap

在JDK1.7中ConcurrentHashMap使用的是分段锁的机制，实现并发的跟新操作，底层采用数组+链表的数据结构

其中包含两个重要的类Segment和HashEntry

1:Segment是继承ReentrantLock用来充当锁的角色，每个Segment对象守护每个散列映射表的若干个桶

2、HashEntry是用来封装键值对的，它包含有四个属性key 、hash、value、next其中key hash和next都是final的，value域是被volatile修饰的，因此HashEntry的对象几乎是不可变的，这是ConcurrentHashMap读操作并不需要加锁的一个重要的原因。

3：每个桶是由若干个HashEntry对象连接起来的链表

一个 ConcurrentHashMap 实例中包含由若干个 Segment 对象组成的数组，下面我们通过一个图来演示一下 ConcurrentHashMap 的结构： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180706/7EbkaggcAh.png?imageslim)

## 三、JDK1.8分析

jdk1.8已经抛弃了Segment分段所机制，利用的是CAS+Synchronized来保证并发更新的安全，底层采用的是数组+链表+红黑树的数据结构

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180706/1Bf74dC5eh.png?imageslim)

### 3.1、重要的概念

在开始正式的源码分析之前，有些重要的概念还需要介绍一下：

1：table:默认为null，初始话发生在第一次插入的操作，默认大小为16的数组，用来存储Node结点数据，扩容为2的幂次方

2：nextTable:默认为null，扩容时新生成的数组，其大小为元数组大小的 两倍

3：**sizeCt**l:这个量不同的值具有不同的含义：

- -1表示的是数组正在进行初始化
- -N表示的是正有N-1个线程正在进行扩容
- 正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小,这一点类似于扩容阈值的概念。还后面可以看到，它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。实际容量>=sizeCtl，则扩容

4**：concurrencyLevel**:

concurrencyLevel，能够同时更新ConccurentHashMap且不产生锁竞争的最大线程数，在Java8之前实际上就是ConcurrentHashMap中的分段锁个数，即Segment[]的数组长度。正确地估计很重要，当低估，数据结构将根据额外的竞争，从而导致线程试图写入当前锁定的段时阻塞；相反，如果高估了并发级别，你遇到过大的膨胀，由于段的不必要的数量; 这种膨胀可能会导致性能下降，由于高数缓存未命中。**在Java8里，仅仅是为了兼容旧版本而保留。唯一的作用就是保证构造map时初始容量不小于concurrencyLevel。** 

5：重要的内部类**Node**：保存key，value及key的hash值的数据结构。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;//volatile类型的
        volatile Node<K,V> next;//volatile类型的


        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        /*
            HashMap中Node类的hashCode()方法中的代码为：Objects.hashCode(key) ^ Objects.hashCode(value)
            而Objects.hashCode(key)最终也是调用了 key.hashCode()，因此，效果一样。写法不一样罢了
        */;
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) { // 不允许修改value值，HashMap允许
            throw new UnsupportedOperationException();
        }
        /*
             HashMap使用if (o == this)，且嵌套if；ConcurrentHashMap使用&& 
             个人觉得HashMap格式的代码更好阅读和理解
        */
        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /*
         * Virtualized support for map.get(); overridden in subclasses.
         *增加find方法辅助get方法  ，HashMap中的Node类中没有此方法
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
```

Node：保存key，value及key的hash值的数据结构。其中value和next都用volatile修饰，保证并发的可见性。 **它增加了find方法辅助map.get()方法。** 

6：TreeNode

- 链表>8，才可能转为TreeNode.
- HashMap的TreeNode继承至LinkedHashMap.Entry；而这里继承自己实现的Node，将带有next指针，便于treebin访问。
- 树节点类，另外一个核心的数据结构。当链表长度过长的时候，会转换为TreeNode。但是与HashMap不相同的是，**它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中，由TreeBin完成对红黑树的包装**。而且TreeNode在ConcurrentHashMap集成自Node类，而并非HashMap中的集成自LinkedHashMap.Entry类，也就是说TreeNode带有next指针，这样做的目的是方便基于TreeBin的访问。

7：TreeBin

- TreeBin用于封装维护TreeNode，包含putTreeVal、lookRoot、UNlookRoot、remove、balanceInsetion、balanceDeletion等方法
- 这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。它代替了TreeNode的根节点**，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象**，这是与HashMap的区别。另外这个类还带有了读写锁。

8：Unsafe与CAS

在ConcurrentHashMap中，随处可以看到Unsafe, 大量使用了U.compareAndSwapXXX的方法，这个方法是利用一个CAS算法实现无锁化的修改值的操作，他可以大大降低锁代理的性能消耗。**这个算法的基本思想就是不断地去比较当前内存中的变量值与你指定的一个变量值是否相等，如果相等，则接受你指定的修改的值，否则拒绝你的操作**。因为当前线程中的值已经不是最新的值，你的修改很可能会覆盖掉其他线程修改的结果。这一点与乐观锁，SVN的思想是比较类似的。

9：3个原子操作

1. tabAt // 获取索引i处Node
2. casTabAt // 利用CAS算法设置i位置上的Node节点（将c和table[i]比较，相同则插入v）。
3. setTabAt // 设置节点位置的值，仅在上锁区被调用

### 3.2：构造函数

先看源码：

```java
  /**
     * Creates a new, empty map with the default initial table size (16).
     */
    public ConcurrentHashMap() {
    }

    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }

    /*
     * Creates a new map with the same mappings as the given map.
     *
     */
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        //private static final int DEFAULT_CAPACITY = 16;
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }

    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        //唯一的作用就是保证构造map时初始容量不小于concurrencyLevel。
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
```

上面的构造函数主要干了两件事：

1、参数的有效性检查

2、table初始化的长度(如果不指定默认情况下为16)。

实例化ConcurrentHashMap时带参数时，会根据参数调整table的大小，假设参数为100，最终会调整成256，确保table的大小总是2的幂次方，算法如下tableSizeFor;

**注意:**ConcurrentHashMap在构造函数中只会初始化sizeCtl值，并不会直接初始化table，而是延缓到第一次put操作。 

### 3.3、table初始化和Put方法

前面已经提到过，table初始化操作会延缓到第一次put行为。但是put是可以并发执行的，Doug Lea是如何实现table只初始化一次的？让我们来看看源码的实现。 

put(K key, V value)方法的功能：将制定的键值对映射到table中，key/value均不能为null 

put方法的代码如下： 

```java
 public V put(K key, V value) {
        return putVal(key, value, false);
    }
```

由于直接是调用了putVal(key, value, false)方法，那就我们就继续看。

putVal(key, value, false)方法的代码如下：

```java
/** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());//计算hash值，两次hash操作
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {//类似于while(true)，死循环，直到插入成功 
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)//检查是否初始化了，如果没有，则初始化
                tab = initTable();
                /*
                    i=(n-1)&hash 等价于i=hash%n(前提是n为2的幂次方).即取出table中位置的节点用f表示。
                    有如下两种情况：
                    1、如果table[i]==null(即该位置的节点为空，没有发生碰撞)，则利用CAS操作直接存储在该位置，如果CAS操作成功则退出死循环。
                    2、如果table[i]!=null(即该位置已经有其它节点，发生碰撞)
                */
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)//检查table[i]的节点的hash是否等于MOVED，如果等于，则检测到正在扩容，则帮助其扩容
                tab = helpTransfer(tab, f);//帮助其扩容
            else {//运行到这里，说明table[i]的节点的hash值不等于MOVED。
                V oldVal = null;
                synchronized (f) {//锁定,（hash值相同的链表的头节点）
                    if (tabAt(tab, i) == f) {//避免多线程，需要重新检查
                        if (fh >= 0) {//链表节点
                            binCount = 1;
                            /*
                            下面的代码就是先查找链表中是否出现了此key，如果出现，则更新value，并跳出循环，
                            否则将节点加入到里阿尼报末尾并跳出循环
                            */
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)//仅putIfAbsent()方法中onlyIfAbsent为true
                                        e.val = value;//putIfAbsent()包含key则返回get，否则put并返回  
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {//插入到链表末尾并跳出循环
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) { //树节点，
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {//插入到树中
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //插入成功后，如果插入的是链表节点，则要判断下该桶位是否要转化为树
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)//实则是>8,执行else,说明该桶位本就有Node
                        treeifyBin(tab, i);//若length<64,直接tryPresize,两倍table.length;不转树 
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

```java
分析上面代码的过程：
/*
putVal(K key, V value, boolean onlyIfAbsent)方法干的工作如下：
1、检查key/value是否为空，如果为空，则抛异常，否则进行2
2、进入for死循环，进行3
3、检查table是否初始化了，如果没有，则调用initTable()进行初始化然后进行2，否则进行4
4、根据key的hash值计算出其应该在table中储存的位置i，取出table[i]的节点用f表示。
    根据f的不同有如下三种情况：1）如果table[i]==null(即该位置的节点为空，没有发生碰撞)，
                                则利用CAS操作直接存储在该位置，如果CAS操作成功则退出死循环。
                                2）如果table[i]!=null(即该位置已经有其它节点，发生碰撞)，碰撞处理也有两种情况
                                    2.1）检查table[i]的节点的hash是否等于MOVED，如果等于，则检测到正在扩容，则帮助其扩容
                                    2.2）说明table[i]的节点的hash值不等于MOVED，如果table[i]为链表节点，则将此节点插入链表中即可
                                        如果table[i]为树节点，则将此节点插入树中即可。插入成功后，进行 5
5、如果table[i]的节点是链表节点，则检查table的第i个位置的链表是否需要转化为数，如果需要则调用treeifyBin函数进行转化


*/
```

1、第一步根据给定的key的hash值找到其在table中的位置index。

2、找到位置index后，存储进行就好了。

只是这里的存储有三种情况罢了，第一种：table[index]中没有任何其他元素，即此元素没有发生碰撞，这种情况直接存储就好了哈。第二种，table[i]存储的是一个链表，如果链表不存在key则直接加入到链表尾部即可，如果存在key则更新其对应的value。第三种，table[i]存储的是一个树，则按照树添加节点的方法添加就好。

在putVal函数，出现了如下几个函数

1、casTabAt tabAt 等CAS操作

2、initTable 作用是初始化table数组

3、treeifyBin 作用是将table[i]的链表转化为树



上面的代码中涉及到的其他的函数：

1：初始化的代码：

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
//如果一个线程发现sizeCtl<0，意味着另外的线程执行CAS操作成功，当前线程只需要让出cpu时间片
        //sizeCtl为负表示的是- -1表示的是数组正在进行初始化 -N表示的是正有N-1个线程正在进行扩容
        if ((sc = sizeCtl) < 0) 
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}

```

sizeCtl默认为0，如果ConcurrentHashMap实例化时有传参数，sizeCtl会是一个2的幂次方的值。所以执行第一次put操作的线程会执行Unsafe.compareAndSwapInt方法修改sizeCtl为-1，有且只有一个线程能够修改成功，其它线程通过Thread.yield()让出CPU时间片等待table初始化完成。

 

 **3个用的比较多的CAS操作：casTabAt tabAt setTabAt**

1. tabAt // 获取索引i处Node
2. casTabAt // 利用CAS算法设置i位置上的Node节点（将c和table[i]比较，相同则插入v）。
3. setTabAt // 设置节点位置的值，仅在上锁区被调用

```java
 /*
        3个用的比较多的CAS操作
    */

    @SuppressWarnings("unchecked") // ASHIFT等均为private static final  
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) { // 获取索引i处Node  
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);  
    }  
    // 利用CAS算法设置i位置上的Node节点（将c和table[i]比较，相同则插入v）。  
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,  
                                        Node<K,V> c, Node<K,V> v) {  
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);  
    }  
    // 设置节点位置的值，仅在上锁区被调用  
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {  
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);  
    }
```

treeifyBin方法：将数组tab的第index位置的链表转化为 树 

```java
/*
     *链表转树：将将数组tab的第index位置的链表转化为 树
     */
    private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)// 容量<64，则table两倍扩容，不转树了
                tryPresize(n << 1);
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                synchronized (b) { // 读写锁  
                    if (tabAt(tab, index) == b) {
                        TreeNode<K,V> hd = null, tl = null;
                        for (Node<K,V> e = b; e != null; e = e.next) {
                            TreeNode<K,V> p =
                                new TreeNode<K,V>(e.hash, e.key, e.val,
                                                  null, null);
                            if ((p.prev = tl) == null)
                                hd = p;
                            else
                                tl.next = p;
                            tl = p;
                        }
                        setTabAt(tab, index, new TreeBin<K,V>(hd));
                    }
                }
            }
        }
    }
```

treeifyBin方法的思想也相当的简单，如下：

1、检查下table的长度是否大于等于MIN_TREEIFY_CAPACITY（64），如果不大于，则调用tryPresize方法将table两倍扩容就可以了，就不降链表转化为树了。如果大于，则就将table[i]的链表转化为树。

 

 ### 3.4:ConcurrentHashMap类的get(int key)方法

看完了ConcurrentHashMap类的put(int key ,int value)方法的内部实现，接着看此类的get(int key)方法 

```java
 /*
     功能：根据key在Map中找出其对应的value，如果不存在key，则返回null，
     其中key不允许为null，否则抛异常
     */
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());//两次hash计算出hash值
        if ((tab = table) != null && (n = tab.length) > 0 &&//table不能为null，是吧
            (e = tabAt(tab, (n - 1) & h)) != null) {//table[i]不能为空，是吧
            if ((eh = e.hash) == h) {//检查头结点
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)//table[i]为一颗树
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {//链表，遍历寻找即可
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

get(int key)方法代码实现流程如下：

1、根据key调用spread计算hash值；并根据计算出来的hash值计算出该key在table出现的位置i.

2、检查table是否为空；如果为空，返回null，否则进行3

3、检查table[i]处桶位不为空；如果为空，则返回null，否则进行4

4、先检查table[i]的头结点的key是否满足条件，是则返回头结点的value；否则分别根据树、链表查询。

### 3.5:table扩容

当table容量不足的时候，即table的元素数量达到容量阈值sizeCtl，需要对table进行扩容。
 整个扩容分为两部分：

1. 构建一个nextTable，大小为table的两倍。
2. 把table的数据复制到nextTable中。

这两个过程在单线程下实现很简单，但是ConcurrentHashMap是支持并发插入的，扩容操作自然也会有并发的出现，这种情况下，第二步可以支持节点的并发复制，这样性能自然提升不少，但实现的复杂度也上升了一个台阶。

先看第一步，构建nextTable，毫无疑问，这个过程只能只有单个线程进行nextTable的初始化，具体实现如下：

 ```java
private final void addCount(long x, int check) {
    ... 省略部分代码
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}

 ```

通过Unsafe.compareAndSwapInt修改sizeCtl值，保证只有一个线程能够初始化nextTable，扩容后的数组长度为原来的两倍，但是容量是原来的1.5。

节点从table移动到nextTable，大体思想是遍历、复制的过程。

 

1. 首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素f，初始化一个forwardNode实例fwd。
2. 如果f == null，则在table中的i位置放入fwd，这个过程是采用Unsafe.compareAndSwapObjectf方法实现的，很巧妙的实现了节点的并发移动。
3. 如果f是链表的头节点，就构造一个反序链表，把他们分别放在nextTable的i和i+n的位置上，移动完成，采用Unsafe.putObjectVolatile方法给table原位置赋值fwd。
4. 如果f是TreeBin节点，也做一个反序处理，并判断是否需要untreeify，把处理的结果分别放在nextTable的i和i+n的位置上，移动完成，同样采用Unsafe.putObjectVolatile方法给table原位置赋值fwd。

遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍 ，扩容完成。

 

 

 

 

 

 

 

 

