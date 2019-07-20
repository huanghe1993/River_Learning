# TreeMap的深入剖析

[TOC]

##  一、简介

`TreeMap`最早出现在`JDK 1.2`中，是 Java 集合框架中比较重要一个的实现。TreeMap 底层基于`红黑树`实现，可保证在`log(n)`时间复杂度内完成 containsKey、get、put 和 remove 操作，效率很高。另一方面，由于 TreeMap 基于红黑树实现，这为 TreeMap 保持键的有序性打下了基础。总的来说，TreeMap 的核心是红黑树，其很多方法也是对红黑树增删查基础操作的一个包装。所以只要弄懂了红黑树，TreeMap 就没什么秘密了。



## 二、概览

`TreeMap`继承自`AbstractMap`，并实现了 `NavigableMap`接口。NavigableMap 接口继承了`SortedMap`接口，SortedMap 最终继承自`Map`接口，同时 AbstractMap 类也实现了 Map 接口。以上就是 TreeMap 的继承体系，描述起来有点乱，不如看图了：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/6cK4mA6dHA.png?imageslim)

- TreeMap实现了NavigableMap接口，而NavigableMap接口继承着SortedMap接口，致使我们的**TreeMap是有序的**！
- TreeMap底层是红黑树，它方法的时间复杂度都不会太高:log(n)~
- 非同步
- 使用Comparator或者Comparable来比较key是否相等与排序的问题~

上图就是 TreeMap 的继承体系图，比较直观。这里来简单说一下继承体系中不常见的接口`NavigableMap`和`SortedMap`，这两个接口见名知意。先说 NavigableMap 接口，NavigableMap 接口声明了一些列具有导航功能的方法，比如： 

```java
/**
 * 返回红黑树中最小键所对应的 Entry
 */
Map.Entry<K,V> firstEntry();

/**
 * 返回最大的键 maxKey，且 maxKey 仅小于参数 key
 */
K lowerKey(K key);

/**
 * 返回最小的键 minKey，且 minKey 仅大于参数 key
 */
K higherKey(K key);

// 其他略
```

通过这些导航方法，我们可以快速定位到目标的 key 或 Entry。

对于SortedMap来说，该类是TreeMap体系中的父接口，也是区别于HashMap体系最关键的一个接口。 

主要原因就是SortedMap接口中定义的第一个方法---Comparator<? super K> comparator()；

该方法决定了TreeMap体系的走向，有了比较器，就可以对插入的元素进行排序了； 

```java
public interface SortedMap<K,V> extends Map<K,V> {
    
    //返回元素比较器。如果是自然顺序，则返回null；
    Comparator<? super K> comparator();
    
    //返回从fromKey到toKey的集合：含头不含尾
    java.util.SortedMap<K,V> subMap(K fromKey, K toKey);

    //返回从头到toKey的集合：不包含toKey
    java.util.SortedMap<K,V> headMap(K toKey);

    //返回从fromKey到结尾的集合：包含fromKey
    java.util.SortedMap<K,V> tailMap(K fromKey);
    
    //返回集合中的第一个元素：
    K firstKey();
   
    //返回集合中的最后一个元素：
    K lastKey();
    
    //返回集合中所有key的集合：
    Set<K> keySet();
    
    //返回集合中所有value的集合：
    Collection<V> values();
    
    //返回集合中的元素映射：
    Set<Map.Entry<K, V>> entrySet();
}
```

TreeMap具有如下特点：

- 不允许出现重复的key；
- 可以插入null键，null值；
- 可以对元素进行排序；
- 无序集合（插入和遍历顺序不一致）；

### 2.1、属性

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/bJiC799FEF.png?imageslim)



##  三、源码分析

`JDK 1.8`中的`TreeMap`源码有两千多行，还是比较多的。本文并不打算逐句分析所有的源码，而是挑选几个常用的方法进行分析。这些方法实现的功能分别是查找、遍历、插入、删除等，其他的方法小伙伴们有兴趣可以自己分析。`TreeMap`实现的核心部分是关于`红黑树`的实现，其绝大部分的方法基本都是对底层红黑树增、删、查操作的一个封装。如简介一节所说，只要弄懂了红黑树原理，TreeMap 就没什么秘密了。关于`红黑树`的原理，请参考文章-[红黑树详细分析](http://www.coolblog.xyz/2018/01/11/%E7%BA%A2%E9%BB%91%E6%A0%91%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90/)，本篇文章不会对此展开讨论。 

### 3.1构造函数

```java
TreeMap() 
          使用键的自然顺序构造一个新的、空的树映射。
TreeMap(Comparator<? super K> comparator) 
          构造一个新的、空的树映射，该映射根据给定比较器进行排序。
TreeMap(Map<? extends K,? extends V> m) 
          构造一个与给定映射具有相同映射关系的新的树映射，该映射根据其键的自然顺序 进行排序。
TreeMap(SortedMap<K,? extends V> m) 
          构造一个与指定有序映射具有相同映射关系和相同排序顺序的新的树映射。
```

这里我们简单的看看TreeMap的使用方式

**（1）使用元素自然排序** 

```java
public class SortedTest implements Comparable<SortedTest> {
    private int age;
    public SortedTest(int age){
        this.age = age;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    //自定义对象，实现compareTo(T o)方法：
    public int compareTo(SortedTest sortedTest) {
        int num = this.age - sortedTest.getAge();
        //为0时候，两者相同：
        if(num==0){
            return 0;
            //大于0时，传入的参数小：
        }else if(num>0){
            return 1;
            //小于0时，传入的参数大：
        }else{
            return -1;
        }
    }

    @Override
    public String toString() {
        return "SortedTest{" +
                "age=" + age +
                '}';
    }
}
```

- 在使用自然排序的时候分为两种情况，第一种是JDK定义的对象，第二种是 我们自己定义的对象
- 在自然顺序的比较中，需要让被比较的 元素实现Comparable接口，否则在向集合中他添加元素时报"java.lang.ClassCastException: com.jiaboyan.collection.map.SortedTest cannot be cast to java.lang.Comparable"异常； 这是因为在调用put()方法时，会将传入的元素转化成Comparable类型对象，所以当你传入的元素没有实现Comparable接口时，就无法转换，遍会报错； 
- 使用JDK定义的对象的时候，JDK是帮我们实现了Compareble接口的自然排序；在使用自定义对象的时候是需要我们自己实现Comparable接口的                  

```java
public class TreeMapTest {
    public static void main(String[] agrs){
        //自然顺序比较
        naturalSort();
    }
    //自然排序顺序：
    public static void naturalSort(){
        //第一种情况：Integer对象,没有实现Comparator使用的是自然排序
        TreeMap<Integer,String> treeMapFirst = new TreeMap<Integer, String>();
        treeMapFirst.put(1,"huanghe");
        treeMapFirst.put(6,"huanghe");
        treeMapFirst.put(3,"huanghe");
        treeMapFirst.put(10,"huanghe");
        treeMapFirst.put(7,"huanghe");
        treeMapFirst.put(13,"huanghe");
        System.out.println(treeMapFirst.toString());

        //第二种情况:SortedTest对象
        TreeMap<SortedTest,String> treeMapSecond = new TreeMap<SortedTest, String>();
        treeMapSecond.put(new SortedTest(10),"huanghe");
        treeMapSecond.put(new SortedTest(1),"huanghe");
        treeMapSecond.put(new SortedTest(13),"huanghe");
        treeMapSecond.put(new SortedTest(4),"huanghe");
        treeMapSecond.put(new SortedTest(0),"huanghe");
        treeMapSecond.put(new SortedTest(9),"huanghe");
        System.out.println(treeMapSecond.toString());
    }

```

**（2）使用自定义比较器排序** 

使用自定义比较器排序，需要在创建TreeMap对象时，将自定义比较器对象传入到TreeMap构造方法中；

自定义比较器对象，需要实现Comparator接口，并实现比较方法compare(T o1,T o2)；

值得一提的是，使用自定义比较器排序的话，被比较的对象无需再实现Comparable接口了；

```java
public class SortedTest {
    private int age;
    public SortedTest(int age){
        this.age = age;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
public class SortedTestComparator implements Comparator<SortedTest> {
    //自定义比较器：实现compare(T o1,T o2)方法：
    public int compare(SortedTest sortedTest1, SortedTest sortedTest2) {
        int num = sortedTest1.getAge() - sortedTest2.getAge();
        if(num==0){//为0时候，两者相同：
            return 0;
        }else if(num>0){//大于0时，后面的参数小：
            return 1;
        }else{//小于0时，前面的参数小：
            return -1;
        }
    }
}

public class TreeMapTest {
    public static void main(String[] agrs){
        //自定义顺序比较
        customSort();
    }
    //自定义排序顺序:
    public static void customSort(){
        TreeMap<SortedTest,String> treeMap = new TreeMap<SortedTest, String>(new SortedTestComparator());
        treeMap.put(new SortedTest(10),"hello");
        treeMap.put(new SortedTest(21),"my");
        treeMap.put(new SortedTest(15),"name");
        treeMap.put(new SortedTest(2),"is");
        treeMap.put(new SortedTest(1),"jiaboyan");
        treeMap.put(new SortedTest(7),"world");
        System.out.println(treeMap.toString());
    }
}
```



### 3.2 查找

`TreeMap`基于红黑树实现，而红黑树是一种自平衡二叉查找树，所以 TreeMap 的查找操作流程和二叉查找树一致。二叉树的查找流程是这样的，先将目标值和根节点的值进行比较，如果目标值小于根节点的值，则再和根节点的左孩子进行比较。如果目标值大于根节点的值，则继续和根节点的右孩子比较。在查找过程中，如果目标值和二叉树中的某个节点值相等，则返回 true，否则返回 false。TreeMap 查找和此类似，只不过在 TreeMap 中，节点（Entry）存储的是键值对`<k,v>`。在查找过程中，比较的是键的大小，返回的是值，如果没找到，则返回`null`。TreeMap 中的查找方法是`get`，具体实现在`getEntry`方法中，相关源码如下：

```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    //如果comparator不为null，调的就是
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    
    // 查找操作的核心逻辑就在这个 while 循环里
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}
```

如果Comparator不为null，接下来我们进去看看`getEntryUsingComparator(Object key)`，是怎么实现的 

```java
 final Entry<K,V> getEntryUsingComparator(Object key) {
        @SuppressWarnings("unchecked")
            K k = (K) key;
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            Entry<K,V> p = root;
            while (p != null) {
                //调用的是Comparator自己实现的方法来回去对应位置，总体的逻辑和外面的
                //没有什么区别
                int cmp = cpr.compare(k, p.key);
                if (cmp < 0)
                    p = p.left;
                else if (cmp > 0)
                    p = p.right;
                else
                    return p;
            }
        }
        return null;
    }
```

### 3.3 遍历

遍历操作也是大家使用频率较高的一个操作，对于`TreeMap`，使用方式一般如下： 

```java
for(Object key : map.keySet()) {
    // do something
}
```

或者

```java
for(Map.Entry entry : map.entrySet()) {
    // do something
}
```

从上面代码片段中可以看出，大家一般都是对 TreeMap 的 key 集合或 Entry 集合进行遍历。上面代码片段中用 foreach 遍历keySet 方法产生的集合，在编译时会转换成用迭代器遍历，等价于： 

```java
Set keys = map.keySet();
Iterator ite = keys.iterator();
while (ite.hasNext()) {
    Object key = ite.next();
    // do something
}
```

另一方面，TreeMap 有一个特性，即可以保证键的有序性，默认是正序。所以在遍历过程中，大家会发现 TreeMap 会从小到大输出键的值。那么，接下来就来分析一下`keySet`方法，以及在遍历 keySet 方法产生的集合时，TreeMap 是如何保证键的有序性的。相关代码如下： 

```java
public Set<K> keySet() {
    return navigableKeySet();
}

public NavigableSet<K> navigableKeySet() {
    KeySet<K> nks = navigableKeySet;
    return (nks != null) ? nks : (navigableKeySet = new KeySet<>(this));
}

static final class KeySet<E> extends AbstractSet<E> implements NavigableSet<E> {
    private final NavigableMap<E, ?> m;
    KeySet(NavigableMap<E,?> map) { m = map; }

    public Iterator<E> iterator() {
        if (m instanceof TreeMap)
            return ((TreeMap<E,?>)m).keyIterator();
        else
            return ((TreeMap.NavigableSubMap<E,?>)m).keyIterator();
    }

    // 省略非关键代码
}

Iterator<K> keyIterator() {
    return new KeyIterator(getFirstEntry());
}

final class KeyIterator extends PrivateEntryIterator<K> {
    KeyIterator(Entry<K,V> first) {
        super(first);
    }
    public K next() {
        return nextEntry().key;
    }
}

abstract class PrivateEntryIterator<T> implements Iterator<T> {
    Entry<K,V> next;
    Entry<K,V> lastReturned;
    int expectedModCount;

    PrivateEntryIterator(Entry<K,V> first) {
        expectedModCount = modCount;
        lastReturned = null;
        next = first;
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Entry<K,V> nextEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSu chElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        // 寻找节点 e 的后继节点
        next = successor(e);
        lastReturned = e;
        return e;
    }

    // 其他方法省略
}
```

上面的代码比较多，keySet 涉及的代码还是比较多的，大家可以从上往下看。从上面源码可以看出 keySet 方法返回的是`KeySet`类的对象。这个类实现了`Iterable`接口，可以返回一个迭代器。该迭代器的具体实现是`KeyIterator`，而 KeyIterator 类的核心逻辑是在`PrivateEntryIterator`中实现的。上面的代码虽多，但核心代码还是 KeySet 类和 PrivateEntryIterator 类的 `nextEntry`方法。KeySet 类就是一个集合，这里不分析了。而 nextEntry 方法比较重要，下面简单分析一下。

在初始化 KeyIterator 时，会将 TreeMap 中包含最小键的 Entry 传给 PrivateEntryIterator。当调用 nextEntry 方法时，通过调用 successor 方法找到当前 entry 的后继，并让 next 指向后继，最后返回当前的 entry。通过这种方式即可实现按正序返回键值的的逻辑。

###3.3、插入

相对于前两个操作，插入操作明显要复杂一些。当往 TreeMap 中放入新的键值对后，可能会破坏红黑树的性质。这里为了描述方便，把 Entry 称为节点。并把新插入的节点称为`N`，N 的父节点为`P`。P 的父节点为`G`，且 P 是 G 的左孩子。P 的兄弟节点为`U`。在往红黑树中插入新的节点 N 后（新节点为红色），会产生下面5种情况： 

1. N 是根节点
2. N 的父节点是黑色
3. N 的父节点是红色，叔叔节点也是红色
4. N 的父节点是红色，叔叔节点是黑色，且 N 是 P 的右孩子
5. N 的父节点是红色，叔叔节点是黑色，且 N 是 P 的左孩子

① 情况说明：被插入的节点是根节点。  

​	处理方法：直接把此节点涂为黑色。

② **情况说明：被插入的节点的父节点是黑色**。  

​	处理方法：什么也不需要做。节点被插入后，仍然是红黑树。

 ③ 情况说明：被插入的节点的父节点是红色。  

​	处理方法：这时就需要分多种情况进行考虑了

上面5中情况中，情况2不会破坏红黑树性质，所以无需处理。情况1 会破坏红黑树性质2（根是黑色），情况3、4、和5会破坏红黑树性质4（每个红色节点必须有两个黑色的子节点）。这个时候就需要进行调整，以使红黑树重新恢复平衡。至于怎么调整，可以参考我另一篇关于红黑树的文章（[红黑树详细分析](http://www.coolblog.xyz/2018/01/11/%E7%BA%A2%E9%BB%91%E6%A0%91%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90/)），这里不再重复说明。接下来分析一下插入操作相关源码： 

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    // 1.如果根节点为 null，将新节点设为根节点
    if (t == null) {
        compare(key, key);
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        // 2.为 key 在红黑树找到合适的位置
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    } else {
        // 与上面代码逻辑类似，省略
    }
    Entry<K,V> e = new Entry<>(key, value, parent);
    // 3.将新节点链入红黑树中
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    // 4.插入新节点可能会破坏红黑树性质，这里修正一下
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

put 方法代码如上，逻辑和二叉查找树插入节点逻辑一致。重要的步骤我已经写了注释，并不难理解。插入逻辑的复杂之处在于插入后的修复操作，对应的方法`fixAfterInsertion`，该方法的源码和说明如下： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/3d6155lglm.png?imageslim)

到这里，插入操作就讲完了。接下来，来说说 TreeMap 中最复杂的部分，也就是删除操作了。 

### 3.4 删除

删除操作是红黑树最复杂的部分，原因是该操作可能会破坏红黑树性质5（从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点），修复性质5要比修复其他性质（性质2和4需修复，性质1和3不用修复）复杂的多。当删除操作导致性质5被破坏时，会出现8种情况。为了方便表述，这里还是先做一些假设。我们把`最终被删除`的节点称为 X，X 的替换节点称为 N。N 的父节点为`P`，且 N 是 P 的左孩子。N 的兄弟节点为`S`，S 的左孩子为 SL，右孩子为 SR。这里特地强调 X 是 `最终被删除` 的节点，是原因**二叉查找树会把要删除有两个孩子的节点的情况转化为删除只有一个孩子的节点的情况**，该节点是欲被删除节点的前驱和后继。

接下来，简单列举一下删除节点时可能会出现的情况，先列举较为简单的情况：

1. 最终被删除的节点 X 是红色节点
2. X 是黑色节点，但该节点的孩子节点是红色

比较复杂的情况：

1. 替换节点 N 是新的根
2. N 为黑色，N 的兄弟节点 S 为红色，其他节点为黑色。
3. N 为黑色，N 的父节点 P，兄弟节点 S 和 S 的孩子节点均为黑色。
4. N 为黑色，P 是红色，S 和 S 孩子均为黑色。
5. N 为黑色，P 可红可黑，S 为黑色，S 的左孩子 SL 为红色，右孩子 SR 为黑色
6. N 为黑色，P 可红可黑，S 为黑色，SR 为红色，SL 可红可黑

上面列举的8种情况中，前两种处理起来比较简单，后6种情况中情况26较为复杂。接下来我将会对情况26展开分析，删除相关的源码如下：

```java
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}

private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    /* 
     * 1. 如果 p 有两个孩子节点，则找到后继节点，
     * 并把后继节点的值复制到节点 P 中，并让 p 指向其后继节点
     */
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    if (replacement != null) {
        /*
         * 2. 将 replacement parent 引用指向新的父节点，
         * 同时让新的父节点指向 replacement。
         */ 
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // 3. 如果删除的节点 p 是黑色节点，则需要进行调整
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // 删除的是根节点，且树中当前只有一个节点
        root = null;
    } else { // 删除的节点没有孩子节点
        // p 是黑色，则需要进行调整
        if (p.color == BLACK)
            fixAfterDeletion(p);

        // 将 P 从树中移除
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```

从源码中可以看出，`remove`方法只是一个简单的保证，核心实现在`deleteEntry`方法中。deleteEntry 主要做了这么几件事：

1. 如果待删除节点 P 有两个孩子，则先找到 P 的后继 S，然后将 S 中的值拷贝到 P 中，并让 P 指向 S
2. 如果最终被删除节点 P（P 现在指向最终被删除节点）的孩子不为空，则用其孩子节点替换掉
3. 如果最终被删除的节点是黑色的话，调用 fixAfterDeletion 方法进行修复

上面说了 replacement 不为空时，deleteEntry 的执行逻辑。上面说的略微啰嗦，如果简单说的话，7个字即可总结：`找后继 -> 替换 -> 修复`。这三步中，最复杂的是修复操作。修复操作要重新使红黑树恢复平衡，修复操作的源码分析如下：

fixAfterDeletion 方法分析如下：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180626/EKLA4dl7kH.png?imageslim)

上面对 fixAfterDeletion 部分代码逻辑就行了分析，通过配图的形式解析了每段代码逻辑所处理的情况。通过图解，应该还是比较好理解的。好了，TreeMap 源码先分析到这里。 

## 总结

对于TreeMap的理解，最重要的是对红黑树要进行深刻的认识

## 参考文献

> [TreeMap源码分析](http://www.coolblog.xyz/2018/01/11/TreeMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/#33-插入)