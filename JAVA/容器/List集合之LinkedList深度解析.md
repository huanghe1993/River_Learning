# List集合之LinkedList深度解析

[TOC]

我们在前面的文章中已经介绍过 **List** 大家族中的 [ArrayList](https://blog.csdn.net/weixin_40304387/article/details/80790177) 和[Vector](https://blog.csdn.net/weixin_40304387/article/details/80792421) 这两位犹如孪生兄弟一般，从底层实现，功能都有着相似之处，除了一些个人行为不同（成员变量，构造函数和方法线程安全）。接下来，我们将会认识一下他们的另一位功能强大的兄弟：**LinkedList**

 ## 一、LinkedList的概览

### 1.1、结构图

首先我们还是看一看LinkedList中的结构图，继承体系关系 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180624/gJIibE9EAc.png?imageslim)

从这个继承体系关系中可以看到LinkeList和ArrayList是有非常大的不同的，LinkedList的 依赖关系如下：

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

仔细的分析依赖关系之前，我们再来看一下Collection集合的总览：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180623/GGDleFgm24.png?imageslim)

1、继承于 AbstractSequentialList ，本质上面与继承 AbstractList 没有什么区别，AbstractSequentialList 完善了 AbstractList 中没有实现的方法。

2、Serializable：成员变量 Node 使用 transient 修饰，通过重写read/writeObject 方法实现序列化。

3、Cloneable：重写clone（）方法，通过创建新的LinkedList 对象，遍历拷贝数据进行对象拷贝。

**4、Deque：实现了Collection 大家庭中的队列接口，说明他拥有作为双端队列的功能。**

LinkedList与ArrayList最大的区别就是LinkedList中实现了Collection中的  **Queue（Deque）接口 拥有作为双端队列的功能**                                                                    

 ###1.2、LinkedList的成员变量

```java
  //当前有多少个结点
  transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     * 第一个结点
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     * 最后一个结点
     */
    transient Node<E> last;
    
    //Node的数据结构
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

```

其中Node的数据结构是：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180624/khbEC4D8ka.png?imageslim)

 LinkedList 的成员变量主要由 size（数据量大小），first(头节点)和last（尾节点）。结合数据结构中双端链表的思想，每个节点需要拥有，保存数据（E item），指向下一节点（Node next ）和指向上一节点（Node prev）。

LinkedList 与ArrayLit、Vector 的成员变量对比中，明显没有提供 **MAX_ARRAY_SIZE** 这一个最大值的限定，这是由于链表没有长度限制的原因，他的内存地址不需要分配固定长度进行存储，只需要记录下一个节点的存储地址即可完成整个链表的连续。

这篇文章的 源码是是基于JDK1.8的，那么LinkedList在JDK1.6与JDK1.8有什么区别呢？

>主要不同为，LinkedList 在1.6 版本以及之前，只通过一个 header 头指针保存队列头和尾。这种操作可以说很有深度，但是从代码阅读性来说，却加深了阅读代码的难度。因此在后续的JDK 更新中，将头节点和尾节点 区分开了。节点类也更名为 Node。

> 为什么Node这个类是静态的？答案是：这跟内存泄露有关，Node类是在LinkedList类中的，也就是一个内部类，若不使用static修饰，那么Node就是一个普通的内部类，在java中，一个普通内部类在实例化之后，默认会持有外部类的引用，这就有可能造成内存泄露。但使用static修饰过的内部类（称为静态内部类），就不会有这种问题，在Android中，有很多这样的情况，如Handler的使用。好像扯远了~

### 1.3、LinkedList 构造函数

LinkedList 只提供了两个构造函数：

- LinkedList()
- LinkedList(Collection<? extends E> c)

在JDK1.8 中，LinkedList 的构造函数 LinkedList（） 是一个空方法，并没有提供什么特殊操作。区别于 JDK1.6 中，会初始化 header 为一个空的指针对象。

####1.3.1、LinkedList()

在**jdk1.6**中的实现是   ：

```java
private transient Entry<E> header = new Entry<E>(null, null, null);
    public LinkedList() {
        header.next = header.previous = header;
    }

```

**JDK 1.8** 在使用的时候，才会创建第一个节点。 

```java
 public LinkedList() {
    }
```

####1.3.2、LinkedList(Collection<? extends E> c)

```java
public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
}
```

#### 1.3.3、小结

LinkedList 在新版本的实现中，除了区分了头节点和尾节点外，更加注重在使用时进行内存分配，这里跟ArrayList 类似（ArrayList 默认构造器是创建一个空的数组对象）。 

 ### 1.4、LinkedList的方法

#### 1.4.1、添加方法（Add）概览

LinkedList 继承了 AbstractSequentialList（AbstractList），同时实现了Deque（队列） 接口，因此，他在添加方法 这一块，包含了两者的操作： 

**AbstractSequentialList**:

- add(E e)
- add(int index，E e)
- addAll(Collection<? extends E> c)
- addAll(int index, Collection<? extends E> c)

 **Deque**

- addFirst(E e)
- addLast(E e)
- offer(E e)
- offerFirst(E e)
- offerLast(E e)

 下面关于上面的方法有以下的几点需要进行说明的是：

> **1、add()和offer()区别:**
>
> add()和offer()都是向队列中添加一个元素。一些队列有大小限制，因此如果想在一个满的队列中加入一个新项，调用 add() 方法就会抛出一个 unchecked 异常，而调用 offer() 方法会返回 false。因此就可以在程序中进行有效的判断！

 #### 1.4.2、 add(E e) & addLast(E e) & offer(E e) & offerLast(E e)

虽然 LinkedList 分别实现了List 和 Deque 的添加方法，但是在某种意义上，这些方法其实都是有共性的。例如，我们调用add（E e） 方法，不管是ArrayList 或 Vector 等列表，都是默认在数组末尾进行添加，因此与 队列中在末尾添加节点 addLast（E e） 是有着一样的韵味的。所以，从LinkedList 的源码中，这几个方法，底层操作其实是一致的。

 ```java
 public boolean add(E e) {
        linkLast(e);
        return true;
    }

public void addLast(E e) {
        linkLast(e);
    }

public boolean offer(E e) {
        return add(e);
    }

public boolean offerLast(E e) {
        addLast(e);
        return true;
    }

 void linkLast(E e) {
        final Node<E> l = last;
     	//Node的构造函数 Node(Node<E> prev, E element, Node<E> next)
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

 ```

我们主要是分析linkLast的代码，也就是链表在末尾进行插入的代码 ：    

-  获取表尾结点
- 创建插入结点，使插入结点的前一个元素指向表尾，数据为 e ,下一个元素指向null
- 更新尾指针，使尾指针指向新插入的结点
- 如果l(初始末尾结点) == null，说明这是插入的第一个元素，则头结点指向新插入的结点
- 如果不是，则更新初始的尾结点，使其next的指针指向新插入的结点 

思考：为什么Node l 需要使用的是final进行修饰的 ？

> 首先我们大概的了解一下final修饰变量的作用，一个永不改变的编译时常量。一个在运行时被初始化的值，而你不希望在运行时改变他

 #### 1.4.3、 addFirst(E e) & offerFirst（E e）

在头部添加元素

```java
public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }
    
    public void addFirst(E e) {
        linkFirst(e);
    }

    private void linkFirst(E e) {
        final Node<E> f = first;
        //Node的构造函数 Node(Node<E> prev, E element, Node<E> next)
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }

```

从上述代码可以看出，offerFirst 和addFirst 其实都是一样的操作，只是返回的数据类型不同。 

下面也是 简单的分析一下linkFirst的步骤：

（1）：获取表头结点f

（2）：创建新结点newNode，新结点的prev指针指向的是null，新结点的尾结点指向的是ｆ（初始的表头结点）

（３）：新的头指针指向新创建的结点newNode

（4）：判断f(初始的表头结点) == null ，如果初始的表头结点为空的情况下，则说明这个链表的初始的状态是空，所以尾指针last指向的也是这个新创建的结点newNode

（5）：如果不为空，初始的表头结点的prev指针指向的是新创建的结点

 #### 1.4.3、add(int index，E e)

这里我们主要讲一下，为什么LinkedList 在添加、删除元素这一方面优于 ArrayList。 

```java
public void add(int index, E element) {
        checkPositionIndex(index);
        // 如果插入节点为末尾，直接插入
        if (index == size)
            linkLast(element);
        // 否则，找到该节点，把新的结点插入到找到的结点的位置
        else
            linkBefore(element, node(index));
    }
    
    Node<E> node(int index) {
        // 这里顺序查找元素，通过二分查找的方式，决定从头或尾节点开始进行查找，时间复杂度为 n/2
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
    
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        //Node的构造函数 Node(Node<E> prev, E element, Node<E> next)
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }

```

LinkedList 在 add（int index，Element e）方法的流程 

- 判断下标有效性
- 如果插入位置为末尾，直接插入
- 否则，遍历1/2的链表找到 index 下标的节点
- 通过 succ 设置新节点的前，后节点

下面分析一下  `linkBefore (E e, Node<E> succ）`的操作：

- 找到待插入结点的前一个结点pred
- 创建需要新插入的结点，新插入结点的prev指针指向的是pred，next指针指向的是找到的结点succ
- 设置succ的prev前向指针指向的是succ
- pred如果为空的话，说明找到的结点是头结点，则头指针指向新创建的结点、
- 如果不是空的情况下，则pred的next指针指向的是新结点

 LinkedList 在插入数据之所以会优于ArrayList，主要是由于在插入数据这一环节（linkBefore），插入计算只需要设置节点的前，后节点即可，而ArrayList 则需要将整个数组的数据进行后移 

 #### 1.4.4、addAll(Collection<? extends E> c)

 LinkedList 中提供的两个addAll 方法中，其实内部实现也是一样的，主要通过： addAll(int index, Collection<? extends E> c)进行实现： 

```java
 public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
    
      public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);
        //将集合转化为数组
        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        //获取插入节点的前节点（prev）和尾节点（next）
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }
        //将集合中的数据编织成链表
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }
        //将 Collection 的链表插入 LinkedList 中。
        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```

####1.4.5、小结

LinkedList 在插入数据优于ArrayList ，主要是因为他只需要修改指针的指向即可，而不需要将整个数组的数据进行转移。而LinkedList 差于没有实现 RandomAccess，或者说 不支持索引搜索的原因，他在查找元素这一操作，需要消耗比较多的时间进行操作（n/2）。

#### 1.4.6、删除方法的总览

**AbstractSequentialList**：

- remove（int index）
- remove（Object o）

**Deque**

- remove()
- removeFirst()
- removeLast()
- removeFirstOccurrence(Object o)
- removeLastOccurrence(Object o）

#### 1.4.7、remove(int index)&remove（Object o）

在 ArrayList 中，remove（Object o） 方法，是通过遍历数组，找到下标后，通过fastRemove（与 remove（int i） 类似的操作）进行删除。而LinkedList，则是遍历链表，找到目标节点（node），通过 unlink 进行删除： 我们这里主要来看看 unlink 方法：

```java
public E remove(int index) {
    checkElementIndex(index);
    //node(index)找到index位置的元素
    return unlink(node(index));
}

/**remove(Object o)这个删除元素的方法的形参o是数据本身，而不是LinkedList集合中的元素（节点），所以需要先通过节点遍历的方式，找到o数据对应的元素，然后再调用unlink(Node x)方法将其删除
*/
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}


E unlink(Node<E> x) {
    // assert x != null;
   	//x的数据域element
    final E element = x.item;
    //x的下一个结点
    final Node<E> next = x.next;
    //x的上一个结点
    final Node<E> prev = x.prev;
	
   	//如果x的上一个结点是空结点的话，那么说明这个结点是头结点
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

在remove(int index)这个方法中，先通过index和node(int index)拿到了要被删除的元素x，然后调用了unlink(Node x)方法将其删除，自然，LinkedList删除元素的核心方法就是unlink(Node x)，删除操作分以下几个步骤：

1、 通过要删除的元素x拿到它的前驱节点prev和后继节点next。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/GILIB0eE4i.png?imageslim)

2、 若前驱节点prev为null，说明x是集合中的首个元素，直接将first指向后继节点next即可； 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/Hfaeab6lmA.png?imageslim)

若不为null，则让前驱节点prev的next指向后继节点next，再将x的prev置空。（这时prev与x的关联就解除了，并与next建立了联系）。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/JCmfdlCcii.png?imageslim)

3、若后继节点next为null，说明x是集合中的最后一个元素，直接将last指向前驱节点prev即可；（下图分别对应步骤2中的两种情况） 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/7ELhIFjAbf.png?imageslim)



若不为null，则让后继节点next的prev指向前驱节点prev，再将x的next置空。（这时next与x的关联就解除了，并与prev建立了联系） 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180625/HB0cBcK5i4.png?imageslim)

 4、最后，让记录集合长度的size减1。

> 说到底就是双向链表的删除擦操作

#### 1.4.8、Deque 中的Remove

Deque 中的 removeFirstOccurrence 和 removeLastOccurrence 主要过程为，首先从first/last 节点开始遍历，当发现第一个目标对象，则调用remove（Object o） 进行删除对象。总体上没有什么特别之处。

 稍有不同的是Deque 中的removeFirst（）和removeLast（）方法，在底层实现上面，由于明确知道删除的对象为first/last对象，因此在删除操作上面 会更加简单： 

 ```java
 public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        //获取到头结点的下一个结点           
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        //头指针指向的是头结点的下一个结点
        first = next;
        //如果next为空，说明这个链表只有一个结点
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
 ```

整体操作为，将first 节点的next 设置为新的头节点，然后将 f 清空。 removeLast 操作也类似。 

 ### 1.5、LinkedList 双端链表（队列Queue）

这里要顺带分析下java中的队列实现，why？因为java中队列的实现就是LinkedList，你可能会疑问，队列的英文是Queue，在java中也有对应的接口，怎么会跟LinkedList扯上关系呢？因为LinkedList实现了队列： 我们之所以说LinkedList 为双端链表，是因为他实现了Deque 接口；我们知道，队列是先进先出的，添加元素只能从队尾添加，删除元素只能从队头删除，Queue中的方法就体现了这种特性。 支持队列的一些操作，我们来看一下有哪些方法实现：

- pop（）是栈结构的实现类的方法，返回的是栈顶元素，并且将栈顶元素删除 
- poll（）是队列的数据结构，获取对头元素并且删除队头元素 
- push（）是栈结构的实现类的方法，把元素压入到栈中
- peek（）获取队头元素 ，但是不删除队列的头元素
- offer（）添加队尾元素 

可以看到Deque 中提供的方法主要有上述的几个方法，接下来我们来看看在LinkedList 中是如何实现这些方法的。

 #### 1.5.1、队列的增

offer（）添加队尾元素 

```java
   public boolean offer(E e) {
       return add(e);
   }
```

具体的实现就是在尾部添加一个元素，我们在上面的代码中已经进行了分析

#### 1.5.2、队列的删

poll（）是队列的数据结构，获取对头元素并且删除队头元素 

```java
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

具体的实现前面已经讲过，删除的是队列头部的元素 

#### 1.5.3、队列的查

peek（）获取队头元素 ，但是不删除队列的头元素

```java
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

#### 1.5.4、栈的增

push（）是栈结构的实现类的方法，把元素压入到栈中

push（） 方法的底层实现，其实就是调用了 addFirst（Object o） 

```JAVA
  public void push(E e) {
       addFirst(e);
   }
```

#### 1.5.5、栈的删

pop（）是栈结构的实现类的方法，返回的是栈顶元素，并且将栈顶元素删除 

 ```java
 public E pop() {
       return removeFirst();
   }
       public E removeFirst() {
       final Node<E> f = first;
       if (f == null)
           throw new NoSuchElementException();
       return unlinkFirst(f);
   }

 ```

## 总结

LinkedList 由于没有实现 RandomAccess，因此，在以随机访问的形式进行遍历时效果会非常低下。除此之外，LinkedList 提供了类似于通过Iterator 进行遍历，节点的prev 或 next 进行遍历，还有for循环遍历，都有不错的效果。

 ## 参考文献

> [Java 集合系列3、骨骼惊奇之LinkedList](https://juejin.im/post/5aeea529f265da0b8e7f4bf2)
>
> [LinkedList与Queue源码分析](https://juejin.im/post/5a02d282f265da43062a3422)

 

 

 

 

 



 

 

 