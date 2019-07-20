# ConcurrentLinkedQueue阻塞队列的实现原理分析

## 1. 引言

在并发编程中我们有时候需要使用线程安全的队列。如果我们要实现一个线程安全的队列有两种实现方式：一种是使用阻塞算法，另一种是使用非阻塞算法。使用阻塞算法的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现，而非阻塞的实现方式则可以使用循环CAS的方式来实现，本文让我们一起来研究下Doug Lea是如何使用非阻塞的方式来实现线程安全队列ConcurrentLinkedQueue的，相信从大师身上我们能学到不少并发编程的技巧。

## 2. ConcurrentLinkedQueue的介绍

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，当我们获取一个元素时，它会返回队列头部的元素。它采用了“wait－free”算法来实现，该算法在Michael & Scott算法上进行了一些修改

## 3. ConcurrentLinkedQueue的结构

我们通过ConcurrentLinkedQueue的类图来分析一下它的结构。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180809/5FLBfKiC82.png?imageslim)

（图1）

ConcurrentLinkedQueue由head节点和tair节点组成，每个节点（Node）由节点元素（item）和指向下一个节点的引用(next)组成，节点与节点之间就是通过这个next关联起来，从而组成一张链表结构的队列。默认情况下head节点存储的元素为空，tair节点等于head节点。

```java
private transient volatile Node<E> tail = head;
```

## 4. 入队列

**入队列就是将入队节点添加到队列的尾部**。为了方便理解入队时队列的变化，以及head节点和tair节点的变化，每添加一个节点我就做了一个队列的快照图。

```java
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    //入队前，创建一个入队节点
    Node<E> n = new Node<E>(e);
    retry:
    //死循环，入队不成功反复入队。
    for (;;) {
        //创建一个指向tail节点的引用
        Node<E> t = tail;
        //p用来表示队列的尾节点，默认情况下等于tail节点。
        Node<E> p = t;
        for (int hops = 0; ; hops++) {
        //获得p节点的下一个节点。
            Node<E> next = succ(p);
        //next节点不为空，说明p不是尾节点，需要更新p后在将它指向next节点
            if (next != null) {
               //循环了两次及其以上，并且当前节点还是不等于尾节点
                if (hops > HOPS && t != tail)
                    continue retry; 
                p = next;
            } 
            //如果p是尾节点，则设置p节点的next节点为入队节点。
            else if (p.casNext(null, n)) {
              //如果tail节点有大于等于1个next节点，则将入队节点设置成tair节点，更新失败了也
没关系，因为失败了表示有其他线程成功更新了tair节点。
                if (hops >= HOPS)
                    casTail(t, n); // 更新tail节点，允许失败
                return true;  
            } 
           // p有next节点,表示p的next节点是尾节点，则重新设置p节点
            else {
                p = succ(p);
            }
        }
    }
}
```

