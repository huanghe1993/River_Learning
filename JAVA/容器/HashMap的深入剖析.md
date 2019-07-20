# HashMap的深入剖析

[TOC]



##前沿

​	之前在实验室一直听师兄们说起的最最最常见的面试笔试的题HashMap，在即将找工作的季节，本着虔诚和认真的态度对这个HashMap做一次非常深入的研究。前面有些文章已经对map的基本知识，散列表的基本知识，数据结构的基本知识做了一下简单而又清晰的介绍！

## 一、HashMap的顶部注释

既然是从头剖析HashMap那么就研究的更加的深入和彻底一些，我们不放过任何细小的地方进行源码的解析，所以首先我们先来看看HashMap的顶部注释，看看官方是如何对于HashMap做的注解

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/895Jimb8jA.png?imageslim)

大概以我拙劣的英语水平来解读一下：

```java

Hash table based implementation of the Map interface.  This implementation provides all of the optional map operations, and permits null values and the null key
*/
```

> 基于哈希表的Map接口实现。 该实现提供了所有的可选的映射操作，并且允许key和value为空                 

- map集合的特点就是将键映射到值的对象。一个映射不能包含重复的键，一个键最多只能映射一个值（**也就是一个键对应的值可以为null**）；**而且键也可以为null**,并且map.put(null, “4”)还会覆盖map.put(null, “2”)这个操作。  

 ```java
 HashMap class is roughly equivalent to Hashtable, except that it is  unsynchronized and permits nulls
 ```
> HashMap和Hashtable没有太大的区别，除了

- **HashMap对象的key、value值均可为null** ， **HahTable对象的key、value值均不可为null。** **且两者的的key值均不能重复，若添加key相同的键值对，后面的value会自动覆盖前面的value，但不会报错** ；
- 还有一个不同点就是在同步上，HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的 

```
This class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.
```

> 这个类不保证map是有序的，也不能保证顺序将随时间保持不变

```java
This implementation provides constant-time performance for the basic operations get and put, assuming the hash function disperses the elements properly among the buckets.
```

> 假设散列函数在桶之间正确的分散元素，HashMap对于get和put操作的时间复杂度为O(1) ( *O(1)* 就是 “constant time” ),

```
Iteration over collection views requires time proportional to the "capacity" of the HashMap instance (the number of buckets) plus its size (the number of key-value mappings)
```

>  迭代集合需要的时间与HashMap实例的“容量”（桶的数量）加上其大小（键值映射的数量）成正比  

```
Thus, it's very important not to set the initial
capacity too high (or the load factor too low) if iteration performance is
 important.
```

> 初始容量太高和负载因子 太低对变量都不好，装填因子=表中的记录数/哈希表的长度,如果装载因子太低了就会使得哈希表的长度变大

```
An instance of HashMap has two parameters that affect its
performance: initial capacity and load factor
```

影响HashMap的性能的因素有两个方面：1、初始容量            2装载因子

> - **Initial capacity** The capacity is **the number of buckets** in the hash table, The initial capacity is simply the capacity at the time the hash table is created.
> - **Load factor** The load factor is **a measure of how full the hash table is allowed to get** before its capacity is automatically increased.

简单的说，Capacity就是buckets的数目，Load factor就是buckets填满程度的最大比例。如果对迭代性能要求很高的话不要把`capacity`设置过大，也不要把`load factor`设置过小。当bucket填充的数目（即hashmap中元素的个数）大于`capacity*load factor`时就需要调整buckets的数目为当前的2倍。 

下面的这张图是对前面注释的大概的说明：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/LC7a1AAEE1.png?imageslim)



## 二、HashMap的依赖关系

先来看看HashMap的继承结构体系：

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable 
```

- 1、AbstractMap：表明它是一个散列表，基于Key-Value 的存储方式
- 2、Cloneable：支持拷贝功能
- 3、Seriablizable：重写了write/readObject，支持序列化

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/0eDA21F5ei.png?imageslim)

从依赖关系上面来看，`HashMap` 并没有 `List` 集合 那么的复杂，主要是因为在迭代上面，HashMap 区别 key-value 进行迭代，而他们的迭代又依赖于keySet-valueSet 进行，因此，虽然依赖关系上面HashMap 看似简单，但是内部的依赖关系更为复杂。

##三、HashMap的数据结构

我们知道，在Java中最常用的两种结构是 **数组** 和 **链表**，几乎所有的数据结构都可以利用这两种来组合实现，HashMap 就是这种应用的一个典型。实际上，HashMap 就是一个 **链表数组**，如下是它数据结构： 

**JDK1.8之前的数据结构：**

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/9djGaBC0AL.png?imageslim)

**JDK1.8的数据结构为：**

JDK1.8之前的HashMap都采用上图的结构，都是基于一个数组和多个单链表，hash值冲突的时候，就将对应节点以链表形式存储。如果在一个链表中查找一个节点时，将会花费O(n)的查找时间，会有很大的性能损失。到了JDK1.8，**当同一个Hash值的节点数不小于8时**，不再采用单链表形式存储，而是采用红黑树，如下图所示：  

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/5k1AkkhkcK.png?imageslim)

 ## 四、HashMap 成员变量

```java
//默认 桶（数组） 容量 16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

//最大容量 2的31次方
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//链表转树 大小
//当添加一个元素被添加到有至少TREEIFY_THRESHOLD个结点的桶中，桶中的链表转化为树形结构
static final int TREEIFY_THRESHOLD = 8;

//树转链表 大小
static final int UNTREEIFY_THRESHOLD = 6;

//最小转红黑树容量
static final int MIN_TREEIFY_CAPACITY = 64;

//存储数据节点
static class Node<K,V> implements Map.Entry<K,V> 

//节点数组
transient Node<K,V>[] table;

//数据容量
transient int size;

//操作次数
transient int modCount;

//扩容大小
int threshold;
```

对比于JDK8之前的HashMap ，成员变量主要的区别在于多了红黑树的相关变量，用于标示我们在什么时候进行 `list` -> `Tree` 的转换。 

再来看一下HashMap的一个内部类Node:

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
    	//链表结构，存储下一个元素
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

我们知道Hash的底层是散列表，而在Java中散列表的实现是通过数组+链表的~ 

## 五、HashMap 构造函数

HashMap  提供了四种构造函数：

- HashMap（）：默认构造函数，参数均使用默认大小
- HashMap（int initialCapacity）：指定初始数组大小
- HashMap（int initialCapacity, float loadFactor）：指定初始数组大小，加载因子
- HashMap（Map<? extends K, ? extends V> m）：创建新的HashMap，并将 `m` 中内容存入HashMap中

主要的工作就是完成容量和加载因子的赋值。Hash表都是采用的懒加载方式，当第一次插入数据时才会创建。 

threshold = capacity * loadFactor 

| 名称            | 用途                                                         |
| --------------- | ------------------------------------------------------------ |
| initialCapacity | HashMap 初始容量                                             |
| loadFactor      | 负载因子                                                     |
| threshold       | 当前 HashMap 所能容纳键值对数量的最大值，超过这个值，则需扩容 |

```java
public HashMap(int initialCapacity, float loadFactor) {
        //初始容量不能小于0
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
    	//初始容量不能大于2^31
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
    	//负载因子=表中的记录数/哈希表的长度  不能小于0
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
        //初始化加载因子                                      loadFactor);
        this.loadFactor = loadFactor;
    	///设置HashMap的容量极限，当HashMap的容量达到该极限时就会进行自动扩容操作
        this.threshold = tableSizeFor(initialCapacity);
    }
/**
  * Returns a power of two size for the given target capacity.
  * tableSizeFor的功能（不考虑大于最大容量的情况）是返回大于输入参数且最接近的2的整数次幂   * 的数。比如10，则返回16。
  */
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }                                             

 public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

 public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

 public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
                                               
                                               
```

有关tableSizeFor的功能的具体的解析可以参考这一篇文章《[Java8 HashMap之tableSizeFor](https://www.cnblogs.com/loading4/p/6239441.html)》

 下面是上面代码中的几个问题：

> 在上面代码的第15行`this.threshold = tableSizeFor(initialCapacity);`这一句话设置HashMap的容量极限，而根据   表中记录数=负载因子*哈希表的长度 ，应该是超过capacity * load factor 的值才对
>
> 但是，请注意，在构造方法中，并没有对table这个成员变量进行初始化，table的初始化被推迟到了put方法中，在put方法中会对threshold重新计算。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/bfA0Cag6L5.png?imageslim)

##六、put()方法

put方法可以说是HashMap的核心，我们来看看一个总的介绍图： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/KFgi6iHbkc.png?imageslim)

```java
  public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

上述方法是我们在开发过程中最常使用到的方法，但是却很少人知道，其实内部真正调用的方法是这个`putVal(hash(key), key, value, false, true)` 方法。这里稍微介绍一下这几个参数： 

- hash 值，用于确定存储位置
- key：存入键值
- value：存入数据
- onlyIfAbsent：是否覆盖原本数据，如果为true 则不覆盖
- evict：table 是否处于创建模式

###6.1、 hash(Object key)

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这里的Hash算法本质上就是三步：取key的hashCode值、高位运算、取模运算。 这里引用一张图，易于大家了解相关机制 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/LC4EABFB47.png?imageslim)

得到key的hashCode,与KeyHashCode的高16位做异或运算；

为什么要这样干呢？？我们一般来说直接将key作为哈希值不就好了吗，做异或运算是干嘛用的？？ 

接下来我们继续的分析，看看putVal方法的代码

```java
 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果hash表为空，或者是长度为0，则调用人resize()方法创建hash表
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //如果hash表中的key对应的桶为空，那么k,v将成为该桶的头结点
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //该桶处已经有头结点，即发生了hash冲突，解决冲突使用的方法是链地址法
        else {
            Node<K,V> e; K k;
            //如果添加的值与头结点相同，将e指向p（此时的p是头结点）
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果与头结点不同，并且桶目前已经是红黑树的状态，则调用putTreeVal()方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //如果桶此时仍在链表阶段
            else {
                //遍历，要比较是否与已有的结点相同
                for (int binCount = 0; ; ++binCount) {
                    //将e指向下一个结点，如果是null，说明链表中没有相同的结点，添加到				   //链表的尾部即可
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果此时的链表的个数达到了8，那么就需要将该桶处链表该成红黑						树，
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果与已有的结点相同，跳出循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //如果有重复的结点需要返回旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                //如果onlyIfAbsent为 false或者是oldValue为null,新值将会替换旧值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                //子类实现
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //是一个全新节点，那么size需要+1
        ++modCount;
        //如果超过了阈值，那么resize()扩大容量
        if (++size > threshold)
            resize();
        //子类实现
        afterNodeInsertion(evict);
        return null;
    }
```

在第19行的代码中：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/5AL8K5K5Hm.png?imageslim)

我们是根据key的哈希值来保存在散列表中的，我们表默认的初始容量是16 ，要放到散列表中，就是0-15的位置上。也就是`tab[i = (n - 1) & hash]` ，可以发现的是：在做`&`运算的时候，仅仅是**后4位有效**~那如果我们key的哈希值高位变化很大，低位变化很小。直接拿过去做`&`运算，这就会导致计算出来的Hash值相同的很多。 

而设计者**将key的哈希值的高位也做了运算(与高16位做异或运算，使得在做&运算时，此时的低位实际上是高位与低位的结合)，这就增加了随机性**，减少了碰撞冲突的可能性！ 

### 6.2、putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)

由于源码篇幅过长，这里我进行分开讲解，可以对照源码进行阅读 

####6.2.1、声明成员变量（第一步）

```java
Node<K,V>[] tab; Node<K,V> p; int n, i;
```

第一部分主要先声明几个需要使用到的成员变量：

- tab：对应table 用于存储数据
- p：我们需要存储的数据，将转化为该对象
- n：数组（table） 长度
- i：数组下标

#### 6.2.2 Table 为 null，初始化Table（第二步）

table 为空说明当前操作为第一次操作，通过上面构造函数的阅读，我们可以了解到，我们并没有对table 进行初始化，因此在第一次put 操作的时候，我们需要先将table 进行初始化。 

```java
if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
```

从上述代码可以看到，table 的初始化和扩容，都依赖于 `resize（）` 方法，在后面我们会对该方法进行详细分析。 

#### 6.2.3 Hash碰撞确认下标（True）

```java
 if ((p = tab[i = (n - 1) & hash]) == null)//该下标没有数据，不会发生碰撞
            tab[i] = newNode(hash, key, value, null);
```

在上一步我们已经确认当前table不为空，然后我们需要计算我们对象需要存储的下标了。

如果该下标中并没有数据，我们只需创建一个新的节点，然后将其存入 `tab[]` 即可。

#### 6.2.4 Hash碰撞确认下标（False）

与上述过程相反，Hash碰撞结果后，发现该下标有保存元素，将其保存到变量 `p = tab[i = (n - 1) & hash]` ，现在 `p` 保存的是目标数组下标中的元素。如上图所示(引用)： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/86baBKki6J.png?imageslim)



##### 6.2.4.1 key 值相同覆盖

在获取到 `p` 后，我们首先判断它的 key 是否与我们这次插入的key 相同，如果相同，我们将其引用传递给 `e` 

```java
if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
```

##### 6.2.4.2 红黑树节点处理

```java
else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```

由于在JDK 8后，会对过长的链表进行处理，即 链表 -> 红黑树，因此对应的节点也会进行相关的处理。红黑树的节点则为TreeNode，因此在获取到`p`后，如果他跟首位元素不匹配，那么他就有可能为红黑树的内容。所以进行`putTreeVal(this, tab, hash, key, value)` 操作。该操作的源码，将会在后续进行细述。

##### 6.2.4.3 链表节点处理

```java
   else {
            //for 循环遍历链表，binCount 用于记录长度，如果过长则进行树的转化
                for (int binCount = 0; ; ++binCount) {
                // 如果发现p.next 为空，说明下一个节点为插入节点
                    if ((e = p.next) == null) {
                        //创建一个新的节点
                        p.next = newNode(hash, key, value, null);
                        //判断是否需要转树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        //结束遍历
                        break;
                    }
                    //如果插入的key 相同，退出遍历
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    //替换 p
                    p = e;
                }
            }
```

链表遍历处理(前提是hash值存在冲突了)，整个过程就是，遍历所有节点，当发现如果存在key 与插入的key 相同，那么退出遍历，否则在最后插入新的节点。判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

##### 6.2.4.5 判断是否覆盖

```java
  if (e != null) { // existing mapping for key
                V oldValue = e.value;
      			//如果onlyIfAbsent为false或者是oldValue为null
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
```

如果 `e` 不为空，说明在校验 key 的hash 值，发现存在相同的 key，那么将会在这里进行判断是否对其进行覆盖。 新值会覆盖旧值，并返回旧值；

#### 6.2.5 容量判断

```java
 if (++size > threshold)
            resize();
```

 如果 `size` 大于 `threshold` 则进行扩容处理。 

 ### 6.2.6、小结

putVal 方法主要做了这么几件事情：

1. 当桶数组 table 为空时，通过扩容的方式初始化 table
2. 查找要插入的键值对是否已经存在，存在的话根据条件判断是否用新值替换旧值
3. 如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
4. 判断键值对数量是否大于阈值，大于的话则进行扩容操作

以上就是 HashMap 插入的逻辑，并不是很复杂，这里就不多说了。接下来来分析一下扩容机制。

 ## 七、Resize()扩容

扩容(resize)就是重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组，就像我们用一个小桶装水，如果想装更多的水，就得换大水桶。 

在 HashMap 中，桶数组的长度均是2的幂，阈值大小为桶数组长度与负载因子的乘积。当 HashMap 中的键值对数量超过阈值时，进行扩容。 

在上面的构造函数，和 `put`过程都有调用过`resize（）` 方法，那么，我们接下来将会分析一下 `resize（）`过程。由于`JDK 8`引入了红黑树，我们先从`JDK 7`开始阅读 `resize（）` 过程。下面部分内容参考美团技术团队的博客《[Java 8系列之重新认识HashMap](https://tech.meituan.com/java-hashmap.html)》

### 7.1、JDK1.7  resize()

在 `JDK 7` 中，扩容主要分为了两个步骤：

- 容器扩展
- 内容拷贝

#### 7.1.1 容器扩展

```java
void resize(int newCapacity) {   //传入新的容量
      Entry[] oldTable = table;    //引用扩容前的Entry数组
      int oldCapacity = oldTable.length;         
      if (oldCapacity == MAXIMUM_CAPACITY) {//扩容前的数组大小如果已经达到最(2^30了
          threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1)，这样以后															    就不会扩容了
          return;
      }
   
     Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
     transfer(newTable);                      //！！将数据转移到新的Entry数组里
     table = newTable;                        //HashMap的table属性引用新Entry数组
     threshold = (int)(newCapacity * loadFactor);//修改阈值
 }

```

#### 7.1.2、 内容拷贝

```java
 void transfer(Entry[] newTable) {
     Entry[] src = table;                   //src引用了旧的Entry数组
     int newCapacity = newTable.length;
     for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
         Entry<K,V> e = src[j];             //取得旧Entry数组的每个元素
         if (e != null) {
             src[j] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
             do {
                 Entry<K,V> next = e.next;
                 int i = indexFor(e.hash, newCapacity); //重新计算每个元素在数组中的位置
                 e.next = newTable[i]; //标记[1]
                 newTable[i] = e;      //将元素放在数组上
                 e = next;             //访问下一个Entry链上的元素
             } while (e != null);
         }
     }
 }
```

#### 7.1.3 扩容过程展示（引用）

下面举个例子说明下扩容过程。假设了我们的hash算法就是简单的用key mod 一下表的大小（也就是数组的长度）。其中的哈希桶数组table的size=2， 所以key = 3、7、5，put顺序依次为 5、7、3。(如果按照JDK1.8的put代码，链表的查如应该是尾插法，最后put的元素在末尾)在mod 2以后都冲突在table[1]这里了。这里假设负载因子 loadFactor=1，即当键值对的实际大小size 大于 table的实际大小时进行扩容。接下来的三个步骤是哈希桶数组 resize成4，然后所有的Node重新rehash的过程。

 ![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/96FJH8Fc2m.png?imageslim)



### 7.2 JDK 8 resize()扩容

由于扩容部分代码篇幅比较长，可以对比着博客与源码进行阅读。 与上述流程相似，`JDK 8`中扩容过程主要分成两个部分：

- 容器扩展
- 内容拷贝

下面我们讲解下JDK1.8做了哪些优化。经过观测可以发现，**我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置**。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果（位运算是&（n-1）& hash(key)）。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/1k5iHgdH0i.png?imageslim)

元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化： 

 ![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/del79hG8lC.png?imageslim)

因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，**只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图：** 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/B0Ci3declb.png?imageslim)

这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。这一块就是JDK1.8新增的优化点。有一点注意区别，JDK1.7中rehash的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置，但是从上图可以看出，JDK1.8不会倒置。有兴趣的同学可以研究下JDK1.8的resize源码，写的很赞 

 #### 7.2.1 容器扩展

```java
    Node<K,V>[] oldTab = table;         //创建一个对象指向当前数组
    int oldCap = (oldTab == null) ? 0 : oldTab.length;      // 获取旧数组的长度
    int oldThr = threshold;                             //获取旧的阀值
    int newCap, newThr = 0;   
  
	//如果 table 不为空，表明已经初始化过了
     if (oldCap > 0) {
         //旧表已经达到了最大容量，不能再大，直接返回旧表
         if (oldCap >= MAXIMUM_CAPACITY) {          
             threshold = Integer.MAX_VALUE;  
                return oldTab;
            }
           //否则，新容量为旧容量2倍，新阈值为旧阈值2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 容器扩容一倍，并且将阀值设置为原来的一倍
                newThr = oldThr << 1; // double threshold   
        }
		//下面的else就是旧表不存在，这次扩容就是构造函数需要初始化一个容量
		//如果就阈值>0，说明构造方法中指定了容量
        else if (oldThr > 0) // initial capacity was placed in threshold
            //如果阀值不为空，那么将新的容量设置为当前阀值
            newCap = oldThr;
		//初始化时没有指定阈值和容量，使用默认的容量16和阈值16*0.75=12
        else {               // zero initial threshold signifies using defaults 
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
		//如果新的阈值为0，则阈值=加载因子*哈希表容量
		if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        
        //// 第二步，创建新数组
	   //更新阈值
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
```

| 条件                       | 覆盖情况                            | 备注                                                         |
| -------------------------- | ----------------------------------- | ------------------------------------------------------------ |
| oldCap > 0                 | 桶数组 table 已经被初始化           |                                                              |
| oldThr > 0                 | threshold > 0，且桶数组未被初始化   | 调用 HashMap(int) 和 HashMap(int, float) 构造方法时会产生这种情况，此种情况下 newCap = oldThr，newThr 在第二个条件分支中算出 |
| oldCap == 0 && oldThr == 0 | 桶数组未被初始化，且 threshold 为 0 | 调用 HashMap() 构造方法会产生这种情况。                      |

这里把`oldThr > 0`情况单独拿出来说一下。在这种情况下，会将 oldThr 赋值给 newCap，等价于`newCap = threshold = tableSizeFor(initialCapacity)`。我们在初始化时传入的 initialCapacity 参数经过 threshold 中转最终赋值给了 newCap。这也就解答了前面提的一个疑问：initialCapacity 参数没有被保存下来，那么它怎么参与桶数组的初始化过程的呢？

嵌套分支：

| 条件                         | 覆盖情况                                      | 备注                                                      |
| ---------------------------- | --------------------------------------------- | --------------------------------------------------------- |
| oldCap >= 2^30               | 桶数组容量大于或等于最大桶容量 2^30           | 这种情况下不再扩容                                        |
| newCap < 2^30 && oldCap > 16 | 新桶数组容量小于最大值，且旧桶数组容量大于 16 | 该种情况下新阈值 newThr = oldThr << 1，移位可能会导致溢出 |

这里简单说明一下移位导致的溢出情况，当 loadFactor小数位为 0，整数位可被2整除且大于等于8时，在某次计算中就可能会导致 newThr 溢出归零。见下图 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/iJ2IHi9IHI.png?imageslim)

分支二：

| 条件        | 覆盖情况                                                     | 备注 |
| ----------- | ------------------------------------------------------------ | ---- |
| newThr == 0 | 第一个条件分支未计算 newThr 或嵌套分支在计算过程中导致 newThr 溢出归零 |      |

#### 7.2.2 内容拷贝

在上述容器扩容结束后，如果发现 `oldTab` 不为空，那么接下来将会进行内容拷贝： 

```java
  if (oldTab != null) {
            //对旧数组进行遍历
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                //如果该桶处存在数据(数组的一个值就是一个桶)
                if ((e = oldTab[j]) != null) {
                    //将旧数组中的内容清空,帮助gc
                    oldTab[j] = null;
                    //如果只有一个节点，直接在新表中赋值
                    if (e.next == null)
                        //通过位运算确定下标
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果 当前节点为红黑树节点，进行红黑树相关处理    
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //如果该桶处仍为链表
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
              // 注意：不是(e.hash & (oldCap-1));而是(e.hash & oldCap)
              // (e.hash & oldCap) 得到的是 元素的在数组中的位置是否需要移动,示例如下
                            // 示例1：
                            // e.hash=10 0000 1010
                            // oldCap=16 0001 0000
                            //   &   =0  0000 0000       比较高位的第一位 0
                            //结论：元素位置在扩容后数组中的位置没有发生改变

                            // 示例2：
                            // e.hash=17 0001 0001
                            // oldCap=16 0001 0000
                            //   &   =1  0001 0000      比较高位的第一位   1
                            //结论：元素位置在扩容后数组中的位置发生了改变，新的下标位置是原下标位置+原数组长度

                            // (e.hash & (oldCap-1)) 得到的是下标位置,示例如下
                            //   e.hash=10 0000 1010
                            // oldCap-1=15 0000 1111
                            //      &  =10 0000 1010

                            //   e.hash=17 0001 0001
                            // oldCap-1=15 0000 1111
                            //      &  =1  0000 0001

                            //新下标位置
                            //   e.hash=17 0001 0001
                            // newCap-1=31 0001 1111    newCap=32
                            //      &  =17 0001 0001    1+oldCap = 1+16

                            //元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：
                            // 0000 0001->0001 0001
                            
                           
                            //高位 与运算，确定索引为原索引
                            if ((e.hash & oldCap) == 0) {
                                // 如果原元素位置没有发生变化
                                if (loTail == null)
                                    loHead = e;// 确定首元素
                                 // 第一次进入时     e   -> aa  ; loHead-> aa
                                else
                                    loTail.next = e;
                                //第二次进入时     loTail-> aa  ;    e  -> bb ;  loTail.next -> bb;而loHead和loTail是指向同一块内存的，所以loHead.next 地址为 bb  
                                //第三次进入时     loTail-> bb  ;    e  -> cc ;  loTail.next 地址为 cc;loHead.next.next = cc
                                loTail = e;
                            }
                            //高位与运算，确认索引为 愿索引+ oldCap
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 将所以设置到对应的位置
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
```

在 JDK 1.8 中，重新映射节点需要考虑节点类型。对于树形节点，需先拆分红黑树再映射。对于链表类型节点，则需先对链表进行分组，然后再映射。需要的注意的是，分组后，组内节点相对位置保持不变。关于红黑树拆分的逻辑将会放在下一小节说明，先来看看链表是怎样进行分组映射的。

我们都知道往底层数据结构中插入节点时，一般都是先通过模运算计算桶位置，接着把节点放入桶中即可。事实上，我们可以把重新映射看做插入操作。在 JDK 1.7 中，也确实是这样做的。但在 JDK 1.8 中，则对这个过程进行了一定的优化，逻辑上要稍微复杂一些。在详细分析前，我们先来回顾一下 hash 求余的过程：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/aDb2hK6Ec4.png?imageslim)

上图中，桶数组大小 n = 16，hash1 与 hash2 不相等。但因为只有后4位参与求余，所以结果相等。当桶数组扩容后，n 由16变成了32，对上面的 hash 值重新进行映射： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/5GKkGCL4fJ.png?imageslim)

扩容后，参与模运算的位数由4位变为了5位。由于两个 hash 第5位的值是不一样，所以两个 hash 算出的结果也不一样。上面的计算过程并不难理解，继续往下分析。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/20dEfLD526.png?imageslim)

假设我们上图的桶数组进行扩容，扩容后容量 n = 16，重新映射过程如下: 

依次遍历链表，并计算节点 `hash & oldCap` 的值。如下图所示 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/AFKHGA0j2g.png?imageslim)

如果值为0，将 loHead 和 loTail 指向这个节点。如果后面还有节点 hash & oldCap 为0的话，则将节点链入 loHead 指向的链表中，并将 loTail 指向该节点。如果值为1的话，则让 hiHead 和 hiTail 指向该节点。完成遍历后，可能会得到两条链表，此时就完成了链表分组： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/BFjC7j88H7.png?imageslim)

最后再将这两条链接存放到相应的桶中，完成扩容。如下图： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/6Cj5fdgheb.png?imageslim)

从上图可以发现，重新映射后，两条链表中的节点顺序并未发生变化，还是保持了扩容前的顺序。以上就是 JDK 1.8 中 HashMap 扩容的代码讲解。另外再补充一下，JDK 1.8 版本上HashMap 扩容效率要高于之前版本。如果大家看过 JDK 1.7 的源码会发现，JDK 1.7 为了防止因 hash 碰撞引发的拒绝服务攻击，在计算 hash 过程中引入随机种子。以增强 hash 的随机性，使得键值对均匀分布在桶数组中。在扩容过程中，相关方法会根据容量判断是否需要生成新的随机种子，并重新计算所有节点的 hash。而在 JDK 1.8 中，则通过引入红黑树替代了该种方式。从而避免了多次计算 hash 的操作，提高了扩容效率。 

## 八、链表树化、红黑树链化与拆分

###8.1、链表树化

JDK 1.8 对 HashMap 实现进行了改进。最大的改进莫过于在引入了红黑树处理频繁的碰撞，代码复杂度也随之上升。比如，以前只需实现一套针对链表操作的方法即可。而引入红黑树后，需要另外实现红黑树相关的操作。红黑树是一种自平衡的二叉查找树，本身就比较复杂。本篇文章中并不打算对红黑树展开介绍，本文仅会介绍链表树化需要注意的地方。至于红黑树详细的介绍，如果大家有兴趣，可以参考我的另一篇文章 《[红黑树](https://blog.csdn.net/weixin_40304387/article/details/80720228)》

链表树化的代码treeifyBin在putVal的操作中就用到了，下面的代码是putVal方法的一部分：

```java
 else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果 binCount的记录数，大于TREEIFY_THRESHOLD树化的阈值
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
```





在展开说明之前，先把树化的相关代码贴出来，如下： 

```java
static final int TREEIFY_THRESHOLD = 8;

/**
 * 当桶数组容量小于该值时，优先进行扩容，而不是树化
 */
static final int MIN_TREEIFY_CAPACITY = 64;

static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
}

/**
 * 将普通节点链表转换成树形节点链表
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 桶数组容量小于 MIN_TREEIFY_CAPACITY，优先进行扩容而不是树化
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    //桶的头结点不为null
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // hd 为头节点（head），tl 为尾节点（tail）
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 将普通节点替换成树形节点
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);  // 将普通链表转成由树形节点链表
        if ((tab[index] = hd) != null)
            // 将树形链表转换成红黑树
            hd.treeify(tab);
    }
}

//将普通的结点转化为树形结点
TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    return new TreeNode<>(p.hash, p.key, p.value, next);
}
```

在扩容过程中，树化要满足两个条件：

1. 链表长度大于等于 TREEIFY_THRESHOLD
2. 桶数组容量大于等于 MIN_TREEIFY_CAPACITY

第一个条件比较好理解，这里就不说了。这里来说说加入第二个条件的原因，个人觉得原因如下：

当桶数组容量比较小时，键值对节点 hash 的碰撞率可能会比较高，进而导致链表长度较长。这个时候应该优先扩容，而不是立马树化。毕竟高碰撞率是因为桶数组容量较小引起的，这个是主因。容量小时，优先扩容可以避免一些列的不必要的树化过程。同时，桶容量较小时，扩容会比较频繁，扩容时需要拆分红黑树并重新映射。所以在桶容量比较小的情况下，将长链表转成红黑树是一件吃力不讨好的事。

回到上面的源码中，我们继续看一下 treeifyBin 方法。该方法主要的作用是将普通链表转成为由 TreeNode 型节点组成的链表，并在最后调用 treeify 是将该链表转为红黑树。TreeNode 继承自 Node 类，所以 TreeNode 仍然包含 next 引用，原链表的节点顺序最终通过 next 引用被保存下来。我们假设树化前，链表结构如下：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/l69IbBE7fi.png?imageslim)

HashMap 在设计之初，并没有考虑到以后会引入红黑树进行优化。所以并没有像 TreeMap 那样，要求键类实现 comparable 接口或提供相应的比较器。但由于树化过程需要比较两个键对象的大小，在键类没有实现 comparable 接口的情况下，怎么比较键与键之间的大小了就成了一个棘手的问题。为了解决这个问题，HashMap 是做了三步处理，确保可以比较出两个键的大小，如下： 

1. 比较键与键之间 hash 的大小，如果 hash 相同，继续往下比较
2. 检测键类是否实现了 Comparable 接口，如果实现调用 compareTo 方法进行比较
3. 如果仍未比较出大小，就需要进行仲裁了，仲裁方法为 tieBreakOrder（大家自己看源码吧）

tie break 是网球术语，可以理解为加时赛的意思，起这个名字还是挺有意思的。

通过上面三次比较，最终就可以比较出孰大孰小。比较出大小后就可以构造红黑树了，最终构造出的红黑树如下：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/9kjj3mh0EE.png?imageslim)

橙色的箭头表示 TreeNode 的 next 引用。由于空间有限，prev 引用未画出。可以看出，链表转成红黑树后，原链表的顺序仍然会被引用仍被保留了（红黑树的根节点会被移动到链表的第一位），我们仍然可以按遍历链表的方式去遍历上面的红黑树。这样的结构为后面红黑树的切分以及红黑树转成链表做好了铺垫，我们继续往下分析。 

### 8.2、红黑树拆分

扩容后，普通节点需要重新映射，红黑树节点也不例外。按照一般的思路，我们可以先把红黑树转成链表，之后再重新映射链表即可。这种处理方式是大家比较容易想到的，但这样做会损失一定的效率。不同于上面的处理方式，HashMap 实现的思路则是上好佳。如上节所说，在将普通链表转成红黑树时，HashMap 通过两个额外的引用 next 和 prev 保留了原链表的节点顺序。这样再对红黑树进行重新映射时，完全可以按照映射链表的方式进行。这样就避免了将红黑树转成链表后再进行映射，无形中提高了效率。

以上就是红黑树拆分的逻辑，下面看一下具体实现吧：

```java
// 红黑树转链表阈值
static final int UNTREEIFY_THRESHOLD = 6;

final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;
    // Relink into lo and hi lists, preserving order
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0;
    /* 
     * 红黑树节点仍然保留了 next 引用，故仍可以按链表方式遍历红黑树。
     * 下面的循环是对红黑树节点进行分组，与上面类似
     */
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;
        if ((e.hash & bit) == 0) {
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
            ++lc;
        }
        else {
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
            ++hc;
        }
    }

    if (loHead != null) {
        // 如果 loHead 不为空，且链表长度小于等于 6，则将红黑树转成链表
        if (lc <= UNTREEIFY_THRESHOLD)
            tab[index] = loHead.untreeify(map);
        else {
            tab[index] = loHead;
            /* 
             * hiHead == null 时，表明扩容后，
             * 所有节点仍在原位置，树结构不变，无需重新树化
             */
            if (hiHead != null) 
                loHead.treeify(tab);
        }
    }
    // 与上面类似
    if (hiHead != null) {
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            tab[index + bit] = hiHead;
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

从源码上可以看得出，重新映射红黑树的逻辑和重新映射链表的逻辑基本一致。不同的地方在于，重新映射后，会将红黑树拆分成两条由 TreeNode 组成的链表。如果链表长度小于 UNTREEIFY_THRESHOLD，则将链表转换成普通链表。否则根据条件重新将 TreeNode 链表树化。举个例子说明一下，假设扩容后，重新映射上图的红黑树，映射结果如下： 

![img](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15161648473103.jpg) 

### 8.3、红黑树链化

前面说过，红黑树中仍然保留了原链表节点顺序。有了这个前提，再将红黑树转成链表就简单多了，仅需将 TreeNode 链表转成 Node 类型的链表即可。相关代码如下： 

```java
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    // 遍历 TreeNode 链表，并用 Node 替换
    for (Node<K,V> q = this; q != null; q = q.next) {
        // 替换节点类型
        Node<K,V> p = map.replacementNode(q, null);
        if (tl == null)
            hd = p;
        else
            tl.next = p;
        tl = p;
    }
    return hd;
}

Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    return new Node<>(p.hash, p.key, p.value, next);
}
```

## 九、删除

HashMap 的删除操作并不复杂，仅需三个步骤即可完成。第一步是定位桶位置，第二步遍历链表并找到键值相等的节点，第三步删除节点。相关源码如下： 

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

//按照hash和key删除节点，如果不存在节点，则返回null
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //如果哈希表不为空并且存在桶与hash值匹配,p为桶中的头节点
    if ((tab = table) != null && (n = tab.length) > 0 &&
        // 1. 定位桶位置
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        //case 1：如果头节点匹配 如果键的值与链表第一个节点相等，则将 node 指向该节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {  
            ///case2：如果头节点不匹配，且头节点是TreeNode，即桶中的结构为红黑树结构              如果是 TreeNode 类型，调用红黑树的查找逻辑定位待删除节点
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
           //case 3:如果头节点不匹配，且头节点是Node，即桶中的结构为链表结构，遍历链表
                // 2. 遍历链表，找到待删除节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        
        // 3. 删除节点，并修复链表或红黑树
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
             //如果节点是TreeNode，使用红黑树的方法
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            ////如果待删除节点是头节点，更改桶中的头节点即可
            else if (node == p)
                tab[index] = node.next;
            //在链表遍历过程中，p代表node节点的前驱节点
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

从上面的代码可以看出，removeNode()方法首先是找到待删除的节点，如果存在待删除节点，接下来再执行删除操作。查询时流程与getNode()方法的流程类似，只不过多了在遍历链表时还需要保存前驱节点，因为后面删除时要用到（单链表结构）。删除节点时就比较简单了，三种情况三种处理方式,分别是：

        1. 如果待删除节点是TreeNode，那么调用removeTreeNode()方法 
        2. 如果待删除节点是Node，并且待删除节点就是头节点，那么将头节点更改为原有节点的下一个节点就可以了 
        3. 如果待删除节点是Node且待删除节点不是头节点，那么将遍历过程中保存的前驱节点p的后继节点设为node的后继节点就可以了 

## 十、HashMap的相关的问题

1： HashMap将对象作为key为什么需要重写equals和hashcode方法 

> 答：如果不重写equals和hashCode方法默认的是继承自父类Object的equals和hashCode方法，Object的equals和HashCode默认比较的是对象的内存地址；加入这个我们创建一个User对象，属性有name和age；
>
> 那么 User user1 = new User("张三"，18)； User user2 = new User("张三"，18)；由于这两个对象从语义的角度上来看是相同的，但是存储到HashMap的时候由于equals和hashCode没有被重写，这两个对象都会被存到HashMap中了，而HashMap中是不能由相同的key的，所以就会出错

2：为什么重写了equals方法还需要重写hashCode呢？

> 答：如果只是重写了equals,而没有重写hashCode，那么上面说的两个对象user1和user2都是会存储在hashMap上的，hashMap的存储过程是先计算hashCode值，找到在数组中的位置，然后遍历链表，如果equals返回为true则用新的value替换旧的value,返回旧的value。所以只是重写了equals而没有写hashCoede是会报错的



## 总结：

hashMap的大概的代码分析了一遍，但是还是有很多的细节没有去看，在链表转红黑树的一块没有去仔细的 研究，后续的过程中，可能会对红黑树操作的这一

## 参考文献

> [HashMap和Hashtable的区别](http://www.importnew.com/7010.html)
>
> [Java 8系列之重新认识HashMap](https://tech.meituan.com/java-hashmap.html)
>
> [Java 集合系列4、家喻户晓之HashMap（上）](https://juejin.im/post/5b092443f265da0dc073d22d)
>
> [HashMap 源码详细分析(JDK1.8)](https://segmentfault.com/a/1190000012926722)