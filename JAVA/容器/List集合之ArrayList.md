# List集合之ArrayList深度解析

[TOC]

现在这篇主要讲List集合的三个子类：

- ArrayList

- - 底层数据结构是数组。线程不安全

- LinkedList

- - 底层数据结构是链表。线程不安全

- Vector

- - 底层数据结构是数组。线程安全

其中ArrayList和LinkedList都是用的非常多的

## 一、ArrayList解析

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180623/Ccm4eba9ba.png?imageslim)

首先，我们来讲解的是ArrayList集合，它是我们用得非常非常多的一个集合~ 

首先，我们来看一下ArrayList的属性： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180623/hi56BIk6ij.png?imageslim)



### 1.1、概览

实现了 RandomAccess 接口，因此支持随机访问，这是理所当然的，因为 ArrayList 是基于数组实现的。 

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

####1.1.1、java.io.Serializable接口的作用

可以发现ArrayList实现了java.io.Serializable接口，该接口主要是在序列化的时候起作用，具体为什么需要序列化，请看我的文章[对象序列化为何要定义serialVersionUID的来龙去脉](https://blog.csdn.net/weixin_40304387/article/details/80784462)

实现的第一个接口RandomAccess的作用：`RandomAccess` 是 `List` 实现所使用的**标记接口**，用来表明其**支持快速（通常是固定时间）随机访问**。此接口的主要目的是允许一般的算法更改其行为，从而在将其应用到随机或连续访问列表时能提供良好的性能。 

####1.1.2、讨论 RandomAccess 的作用。 

> 如果 `List` 子类实现了 `RandomAcce ss` 接口，那表示它能快速随机访问存储的元素， 这时候你想到的可能是数组， 通过下标 `index` 访问， 实现了该接口的 `ArrayList` 底层实现就是数组， 同样是通过下标访问， 只是我们需要用 `get()` 方法的形式 ， `ArrayList` 底层仍然是数组的访问形式。同时你应该想到链表， `LinkedList` 底层实现是链表， `LinkedList` 没有实现 `RandomAccess` 接口，发现这一点就是突破问题的关键点。数组支持随机访问， 查询速度快， 增删元素慢； 链表支持顺序访问， 查询速度慢， 增删元素快。所以对应的 `ArrayList` 查询速度快，`LinkedList` 查询速度慢， **`RandomAccess` 这个标记接口就是标记能够随机访问元素的集合， 简单来说就是底层是数组实现的集合。**

为了提升性能，在遍历集合前，我们便可以通过 `instanceof` 做判断， 选择合适的集合遍历方式，当数据量很大时， 就能大大提升性能。

随机访问列表使用循环遍历，顺序访问列表使用迭代器遍历。

先看看 `RandomAccess` 的使用方式

```java
import java.util.*;
public class RandomAccessTest {
    public static void traverse(List list){

        if (list instanceof RandomAccess){
            System.out.println("实现了RandomAccess接口，不使用迭代器");

            for (int i = 0;i < list.size();i++){
                System.out.println(list.get(i));
            }

        }else{
            System.out.println("没实现RandomAccess接口，使用迭代器");

            Iterator it = list.iterator();
            while(it.hasNext()){
                System.out.println(it.next());
            }

        }
    }
    public static void main(String[] args) {
        List<String> arrayList = new ArrayList<>();
        arrayList.add("a");
        arrayList.add("b");
        traverse(arrayList);

        List<String> linkedList = new LinkedList<>();
        linkedList.add("c");
        linkedList.add("d");
        traverse(linkedList);
    }
}
```

**结论**

根据结果我们可以得出结论： `ArrayList` 使用 for 循环遍历优于迭代器，遍历 `LinkedList` 使用 迭代器遍历优于 for 循环遍历

根据以上结论便可利用 `RandomAccess` 在遍历前进行判断，根据 `List` 的不同子类选择不同的遍历方式， 提升算法性能。

####1.1.3、 Cloneable接口的作用：

它允许在堆中克隆出一块和原对象一样的对象，并将这个对象的地址赋予新的引用，这样显然我对新引用的操作，不会影响到原对象。 

**Cloneable接口和Object的clone()方法** 

Java中实现了Cloneable接口的类有很多，像我们熟悉的ArrayList、Calendar、Date、HashMap、Hashtable、HashSet、LinkedList等等。 

**1、Cloneable接口** 

（1）此类实现了Cloneable接口，以指示Object的clone()方法**可以合法地对该类实例进行按字段复制**

（2）如果在**没有实现Cloneable接口的实例上调用Object的clone()方法，则会导致抛出CloneNotSupporteddException**

（3）按照惯例，实现此接口的类应该**使用公共方法重写Object的clone()方法**，Object的clone()方法是一个受保护的方法

**2、Object的clone()方法**

创建并返回此对象的一个副本。对于任何对象x，表达式：

（1）**x.clone() != x为true**（指向的是堆内存中的不同的区域）

（2）**x.clone().getClass() == x.getClass()为true**（堆内存中的内容是相同的）

（3）x.clone().equals(x)一般情况下为true，但这并不是必须要满足的要求

**浅克隆和深克隆** 

浅克隆（shallow clone）和深克隆（deep clone）反映的是，当对象中还有对象的时候，那么：

1、浅克隆，即很表层的克隆，如果我们要克隆对象，只克隆它自身以及它所包含的所有**对象的引用地址**

2、深克隆，克隆除自身对象以外的所有对象，包括自身所包含的**所有对象实例、**

那其实Object的clone()方法，提供的是一种**浅克隆**的机制，如果想要实现对对象的深克隆，在不引入第三方jar包的情况下，可以使用两种办法：

1、先对对象进行序列化，紧接着马上反序列化出

2、先调用super.clone()方法克隆出一个新对象来，然后在子类的clone()方法中手动给克隆出来的非基本数据类型（引用类型）赋值，比如ArrayList的clone()方法：

```java
public Object clone() {
try {
    //1、第一句，浅拷贝出一个ArrayList，此时对象的引用地址还是原来的
    ArrayList<E> v = (ArrayList<E>) super.clone();
    //2、第二句，除了自身外，把Object[] elementData里面的对象全部拷贝一次，做了一次深拷贝(具体的原理需要深入的研究copyof方法的源码)
    v.elementData = Arrays.copyOf(elementData, size);
    v.modCount = 0;
    return v;
} catch (CloneNotSupportedException e) {
    // this shouldn't happen, since we are Cloneable
    throw new InternalError();
}
}
```

#### 1.1.4、数组的默认大小

```java
private static final int DEFAULT_CAPACITY = 10;
```

### 1.2、ArrayList的属性

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable {  
    // 序列化id  
    private static final long serialVersionUID = 8683452581122892189L;  
    // 默认初始的容量  
    private static final int DEFAULT_CAPACITY = 10;  
    // 一个空对象（为什么是new Object[0]呢？）
    //用Object[0]来代替null 很多时候我们需要传递参数的类型，而不是传null，所以用Object[0]
    private static final Object[] EMPTY_ELEMENTDATA = new Object[0];  
    // 一个空对象，如果使用默认构造函数创建，则默认对象内容默认是该值  
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];  
    // 当前数据对象存放地方，当前对象不参与序列化(主要是关键字transient起作用的)  
    transient Object[] elementData;  
    // 当前数组长度  
    private int size;  
    // 数组最大长度  
    private static final int MAX_ARRAY_SIZE = 2147483639;  
  
    // 省略方法。。  
}  
```

关于属性中的 有几点需要说明的是：

1：new Object[0]表示的意思：new Object[0]表示的是创建一个长度为0的Object类型的数组，也就是一个空的数组，为什么不用null表示的，因为很多的情况下，我们在使用的时候需要传入的是参数的类型，而不是传递的是null，所以使用的是new Object[0]

2：EMPTY_ELEMENTDATA和DEFAULTCAPACITY_EMPTY_ELEMENTDATA两者之间的区别：如果使用的是默认的构造函数创建的对象，则返回DEFAULTCAPACITY_EMPTY_ELEMENTDATA；如果是用户在指定容量的大小为0的时候返回的，则返回的是EMPTY_ELEMENTDATA

### 1.3、ArrayList构造函数

#### 1.3.1、无参构造函数

如果不传入参数，则使用默认无参构建方法创建ArrayList对象，如下：

```java
public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}  
```

注意：此时我们创建的ArrayList对象中的elementData中的长度是1，size是0,**当进行第一次add的时候，elementData将会变成默认的长度：10.** 

```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  //确认内部容量
        elementData[size++] = e;
        return true;
    }

 private void ensureCapacityInternal(int minCapacity) {
        // 如果elementData 指向的是 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 的地址
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            //置默认大小 为DEFAULT_CAPACITY
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        //确定实际容量
        ensureExplicitCapacity(minCapacity);
    }


private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 如果超出了容量，进行扩展
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
    	//右移运算符等价于除以2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

上述代码块比较长，这里做个简单的总结：
1、add（E e）：添加元素，首先会判断 elementData 数组的长度，然后设置值
2、ensureCapacityInternal（int minCapacity）：判断 element 是否为空，如果是，则设置默认数组长度
3、ensureExplicitCapacity（int minCapacity）：判断预期增长数组长度是否超过当前容量，如果过超过，则调用grow（）
4、grow(int minCapacity)：对数组进行扩展，默认的长度为10
```

#### 1.3.2、带int类型的构造函数

如果传入参数，则代表指定ArrayList的初始数组长度，传入参数如果是大于等于0，则使用用户的参数初始化，如果用户传入的参数小于0，则抛出异常，构造方法如下： 

```java
public ArrayList(int initialCapacity) {  
    if (initialCapacity > 0) {  
        this.elementData = new Object[initialCapacity];  
    } else if (initialCapacity == 0) {  
        this.elementData = EMPTY_ELEMENTDATA;  
    } else {  
        throw new IllegalArgumentException("Illegal Capacity: "+  
                                           initialCapacity);  
    }  
} 
```

注意在这里就可以看出来EMPTY_ELEMENTDATA和DEFAULTCAPACITY_EMPTY_ELEMENTDATA的区别，EMPTY_ELEMENTDATA是在带int类型的构造函数传入进来的int的数值为0的时候调用的。

>这里就有一个问题值得我们进行深入的思考，为什么这里当int 为0的时候创建的空的数组使用的是**EMPTY_ELEMENTDATA**而不使用**DEFAULTCAPACITY_EMPTY_ELEMENTDATA**？
>
>
>我自己的解答：**DEFAULTCAPACITY_EMPTY_ELEMENTDATA**是一个空的数组实例，和EMPTY_ELEMENTDATA空数组相比是用于了解添加元素时数组膨胀多少
>

#### 1.3.3、ArrayList（Collection c）

将`Collection<T> c` 中保存的数据，首先转换成数组形式（toArray（）方法），然后判断当前数组长度是否为0，为 0 则只想默认数组（EMPTY_ELEMENTDATA）；否则进行数据拷贝。 

```java
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

可能有部分的人对于Arrays.copyof不是很了解，下面我把一段copyOf的源码拿出来，大概一看就能明白的：

```java
public static <T> T[] copyOf(T[] original, int newLength) {  
    return (T[]) copyOf(original, newLength, original.getClass());  
}  
  
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {  
    T[] copy = ((Object)newType == (Object)Object[].class)  
        ? (T[]) new Object[newLength]
        // 通过Java反射来处理数组需要用到java.lang.reflect.Array类。不要和Java集合中的java.util.Arrays类搞混淆了,数组类型可以通过getComponentType()获取的数组的class对象
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);  
    System.arraycopy(original, 0, copy, 0,  
                     Math.min(original.length, newLength));  
    return copy;  
}  
```

#### 1.3.4 总结

上述的三个构造方法可以看出，其实每个构造器内部做的事情都不一样，特别是默认构造器与 ArrayList(int initialCapacity) 这两个构造器直接的区别 ，我们是需要做一些区别的。

- ArrayList（）：指向 **DEFAULTCAPACITY_EMPTY_ELEMENTDATA**，当列表使用的时候，才会进行初始化，会通过判断是不是 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 这个对象而设置数组默认大小。
- ArrayList(int initialCapacity)：当 initialCapacity >0 的时候，设置该长度。如果 initialCapacity =0，则指向 **EMPTY_ELEMENTDATA** 在使用的时候，并不会设置默认数组长度 。

因此 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 与 EMPTY_ELEMENTDATA 的本质区别就在于，会不会设置默认的数组长度。

 

### 1.4、ArrayList的方法

#### 1.4.1、Add方法概览

add方法可以说是ArrayList比较重要的方法了，我们来总览一下： 

ArrayList 添加了四种添加方法：

- add(E  element)
- add(int i , E element)
- addAll(Collection<? extends E> c)
- addAll(int index, Collection<? extends E> c)

 #### 1,4,2、add(E  element)

首先直接的看源码：

```java
   /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

我们再一次的来看看ensureCapacityInternal(size + 1)方法

```java
 private void ensureCapacityInternal(int minCapacity) {
        // 如果elementData 指向的是 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 的地址
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            //置默认大小 为DEFAULT_CAPACITY
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        //确定实际容量
        ensureExplicitCapacity(minCapacity);
    }


private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 如果超出了容量，进行扩展
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
    	//右移运算符等价于除以2，如果第一次是10，扩容之后的大小是15
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

结合默认构造器或其他构造器中，如果默认数组为空，则会在 ensureCapacityInternal（）方法调用的时候进行数组初始化。这就是为什么默认构造器调用的时候，我们创建的是一个空数组，但是在注释里却介绍为 长度为10的数组。 

第二种就是已经存在元素了，当添加第11个元素的时候，minCapacity为11，此时的数组的长度为10，所以minCapacity - elementData.length > 0，需要进行扩容，int newCapacity = oldCapacity + (oldCapacity >> 1)如果第一次是10，扩容之后的大小是15，然后在再把之前的元素进行复制到新的数组中去 elementData = Arrays.copyOf(elementData, newCapacity);

第一次扩容后，如果容量还是小于minCapacity（需要的最小的容量），就将容量扩充为minCapacity。 

 #### 1.4.2、add(int index, E element)

按照惯例还是先查看源码：

```java
public void add(int index, E element) {
    // 判断index 是否有效
        rangeCheckForAdd(index);
    // 计数+1，并确认当前数组长度是否足够
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index); //将index 后面的数据都往后移一位
        elementData[index] = element; //设置目标数据
        size++;
    }

/**
 * A version of rangeCheck used by add and addAll.
 */
private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
} 

/**
  * Constructs an IndexOutOfBoundsException detail message.
  * Of the many possible refactorings of the error handling code,
  * this "outlining" performs best with both server and client VMs.
  */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
} 


```


System提供了一个静态方法arraycopy(),`System.arraycopy`，JVM 提供的数组拷贝实现。这种方式为什么是非常的高效的我们可以参考文章[《System.arraycopy为什么快》](https://blog.csdn.net/wangyangzhizhou/article/details/79504818)  我们可以使用它来实现数组之间的复制。其函数原型是：

```java
@HotSpotIntrinsicCandidate
public static void (Object src,
                             int srcPos,
                             Object dest,
                             int destPos,
                             int length)
src:源数组；	 srcPos:源数组要复制的起始位置；
dest:目的数组；	destPos:目的数组放置的起始位置；	length:复制的长度。
注意：src and dest都必须是同类型或者可以进行转换类型的数组．
```

@HotSpotIntrinsicCandidate这个注解是 HotSpot VM 标准的注解，被它标记的方法表明它为 HotSpot VM 的固有方法， HotSpot VM 会对其做一些增强处理以提高它的执行性能，比如可能手工编写汇编或手工编写编译器中间语言来替换该方法的实现。虽然这里被声明为 native 方法，但是它跟 JDK 中其他的本地方法实现地方不同，固有方法会在 JVM 内部实现，而其他的会在 JDK 库中实现。在调用方面，由于直接调用 JVM 内部实现，不走常规 JNI lookup，所以也省了开销。 

####1.4.3 、addAll(Collection<? extends E> c)

国际惯例，先看源码 

```java
public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
```

addAll() 方法，通过将collection 中的数据转换成 Array[] 然后添加到elementData 数组，从而完成整个集合数据的添加。在整体上没有什么特别之初，这里的collection 可能会抛出控制异常 NullPointerException  需要注意一下。

 #### 1.4.4、 addAll(int index，Collection<? extends E> c)

国际惯例，先看源码 

```java
 public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
}
```

与上述方法相比，这里主要多了两个步骤，判断添加数据的位置是不是在末尾，如果在中间，则需要先将数据向后移动 collection 长度 的位置。 

 #### 1.4.5、删除方法（Remove）

ArrayList 中提供了 五种删除数据的方式：

- remove（int i）
- remove（E element）
- removeRange（int start,int end）
- clear（）
- removeAll（Collection c）

 #### 1.4.6、remove（int i）:

删除数据并不会更改数组的长度，只会将数据从数组中移除，如果目标没有其他有效引用，则在GC 时会进行回收。 

国际惯例查看源码：

```java
public E remove(int index) {
        rangeCheck(index); // 判断索引是否有效
        modCount++;
        E oldValue = elementData(index);  // 获取对应数据
        int numMoved = size - index - 1;  // 判断删除数据位置
        if (numMoved > 0) //如果删除数据不是最后一位，则需要移动数组
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // 让指针最后指向空，进行垃圾回收
        return oldValue;
    }
```

#### 1.4.7、remove（E element）:

这种方式，会在内部进行 AccessRandom 方式遍历数组，当匹配到数据跟 Object 相等，则调用 fastRemove（） 进行删除 

```java
public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
 
```

fastRemove( ): fastRemove 操作与上述的根据下标进行删除其实是一致的。 

```java
 private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

#### 1.4.8、removeRange（int fromIndex, int toIndex）

该方法主要删除了在范围内的数据，通过System.arraycopy 对整部分的数据进行覆盖即可。 

```java
protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

```

#### 1.4.9、removeAll（Collection c）

```java
 private boolean batchRemove(Collection<?> c, boolean complement) {
        //获取数组指针
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                //根据 complement 进行判断删除或留下
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // 进行数据整理
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```

#### 1.4.10、trimToSize()

删除元素时不会减少容量，**若希望减少容量则调用trimToSize()** 

```java
/**
     * Trims the capacity of this <tt>ArrayList</tt> instance to be the
     * list's current size.  An application can use this operation to minimize
     * the storage of an <tt>ArrayList</tt> instance.
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```

#### 1.4.11、toArray（）

ArrayList提供了2个toArray()函数：

- Object[] toArray()
-  T[] toArray(T[] contents)

调用 toArray() 函数会抛出“java.lang.ClassCastException”异常，但是调用 toArray(T[] contents) 能正常返回 T[]。

toArray() 会抛出异常是因为 toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(如如，将Object[]转换为的Integer[])则会抛出“java.lang.ClassCastException”异常，因为Java不支持向下转型。

toArray（） 源码：

```java
 public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
 }
    
```

 ### 1.5、ArrayList线程安全问题的研究

先来回顾一下上面讲的ArrayList添加元素的操作：

```java
public boolean add(E e) {

    /**
     * 添加一个元素时，做了如下两步操作
     * 1.判断列表的capacity容量是否足够，是否需要扩容
     * 2.真正将元素放在列表的元素数组里面
     */
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

这样也就出现了第一个导致线程不安全的隐患，在多个线程进行add操作时可能会导致elementData数组越界。具体逻辑如下： 

1. 列表大小为9，即size=9
2. 线程A开始进入add方法，这时它获取到size的值为9，调用ensureCapacityInternal方法进行容量判断。
3. 线程B此时也进入add方法，它获取到size的值也为9，也开始调用ensureCapacityInternal方法。
4. 线程A发现需求大小为10，而elementData的大小就为10，可以容纳。于是它不再扩容，返回。
5. 线程B也发现需求大小为10，也可以容纳，返回。
6. 线程A开始进行设置值操作， elementData[size++] = e 操作。此时size变为10。
7. 线程B也开始进行设置值操作，它尝试设置elementData[10] = e，而elementData没有进行过扩容，它的下标最大为9。于是此时会报出一个数组越界的异常ArrayIndexOutOfBoundsException.

另外第二步 elementData[size++] = e 设置值的操作同样会导致线程不安全。从这儿可以看出，这步操作也不是一个原子操作，它由如下两步操作构成：  

 ```java
elementData[size] = e;
size = size + 1;
 ```

1. 列表大小为0，即size=0
2. 线程A开始添加一个元素，值为A。此时它执行第一条操作，将A放在了elementData下标为0的位置上。
3. 接着线程B刚好也要开始添加一个值为B的元素，且走到了第一步操作。此时线程B获取到size的值依然为0，于是它将B也放在了elementData下标为0的位置上。
4. 线程A开始将size的值增加为1
5. 线程B开始将size的值增加为2

 这样线程AB执行完毕后，理想中情况为size为2，elementData下标0的位置为A，下标1的位置为B。而实际情况变成了size为2，elementData下标为0的位置变成了B，下标1的位置上什么都没有。并且后续除非使用set方法修改此位置的值，否则将一直为null，因为size为2，添加元素时会从下标为2的位置上开始。 

下面是一个简单的事例来说明这个问题：

```java
public static void main(String[] args) throws InterruptedException {
    final List<Integer> list = new ArrayList<Integer>();

    // 线程A将0-1000添加到list
    new Thread(new Runnable() {
        public void run() {
            for (int i = 0; i < 1000 ; i++) {
                list.add(i);

                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }).start();

    // 线程B将1000-2000添加到列表
    new Thread(new Runnable() {
        public void run() {
            for (int i = 1000; i < 2000 ; i++) {
                list.add(i);

                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }).start();

    Thread.sleep(1000);

    // 打印所有结果
    for (int i = 0; i < list.size(); i++) {
        System.out.println("第" + (i + 1) + "个元素为：" + list.get(i));
    }
}
```

部分的结果：

```java
第7个元素为：3
第8个元素为：1003
第9个元素为：4
第10个元素为：1004
第11个元素为：null
第12个元素为：1005
第13个元素为：6
```

### 1.6、ConcurrentModificationException 

####1.6.1、异常出现的原因

Vector、ArrayList在迭代的时候如果同时对其进行修改就会抛出java.util.ConcurrentModificationException异常。下面我们就来讨论以下这个异常出现的原因以及解决办法。 

先看一段代码：

```java
public class test {
    public static void main(String[] args)  {
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(2);
        Iterator<Integer> iterator = list.iterator();
        while(iterator.hasNext()){
            Integer integer = iterator.next();
            if(integer==2)
                list.remove(integer);
        }
    }
}
```

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180624/IhI2kAd22l.png?imageslim)

从异常信息可以发现，异常出现在checkForComodification()方法中。

　　我们先不看checkForComodification()方法的具体实现，我们先根据程序的代码一步一步看ArrayList源码的实现：

　　首先看ArrayList的iterator()方法的具体实现，查看源码发现在ArrayList的源码中并没有iterator()这个方法，那么很显然这个方法应该是其父类或者实现的接口中的方法，我们在其父类AbstractList中找到了iterator()方法的具体实现，下面是其实现代码：

```java
  public Iterator<E> iterator() {
        return new Itr();
}
```

从这段代码可以看出返回的是一个指向Itr类型对象的引用，我们接着看Itr的具体实现，在AbstractList类中找到了Itr类的具体实现，它是AbstractList的一个成员内部类，下面这段代码是Itr类的所有实现： 

```java
private class Itr implements Iterator<E> {
    /**
     * Index of element to be returned by subsequent call to next.
     */
    int cursor = 0;//表示下一个要访问的元素的索引
   
    /**
	 * Index of element returned by most recent call to next or
     * previous.  Reset to -1 if this element is deleted by a call
     * to remove.
     */
    int lastRet = -1;//表示上一个访问的元素的索引
    int expectedModCount = modCount; //表示对ArrayList修改次数的期望值，它的初始值为modCount。
    public boolean hasNext() {
           return cursor != size();
    }
    public E next() {
           checkForComodification();
        try {
        E next = get(cursor);
        lastRet = cursor++;
        return next;
        } catch (IndexOutOfBoundsException e) {
        checkForComodification();
        throw new NoSuchElementException();
        }
    }
    public void remove() {
        if (lastRet == -1)
        throw new IllegalStateException();
           checkForComodification();
 
        try {
        AbstractList.this.remove(lastRet);
        if (lastRet < cursor)
            cursor--;
        lastRet = -1;
        expectedModCount = modCount;
        } catch (IndexOutOfBoundsException e) {
        throw new ConcurrentModificationException();
        }
    }
 
    final void checkForComodification() {
        if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    }
}
```

　　这里是非常关键的地方：首先在next()方法中会调用checkForComodification()方法，然后根据cursor的值获取到元素，接着将cursor的值赋给lastRet，并对cursor的值进行加1操作。初始时，cursor为0，lastRet为-1，那么调用一次之后，cursor的值为1，lastRet的值为0。注意此时，modCount为0，expectedModCount也为0。 

　　接着往下看，程序中判断当前元素的值是否为2，若为2，则调用list.remove()方法来删除该元素。 

　　我们回顾一下在ArrayList中的remove()方法做了什么： 

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
 
 
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                numMoved);
    elementData[--size] = null; // Let gc do its work
}
```

　　在fastRemove()方法中，首先对modCount进行加1操作（因为对集合修改了一次），然后接下来就是删除元素的操作，最后将size进行减1操作，并将引用置为null以方便垃圾收集器进行回收工作 

　　那么注意此时各个变量的值：对于iterator，其expectedModCount为0，cursor的值为1，lastRet的值为0。 

　　对于list，其modCount为1，size为0；

​	接着看程序代码，执行完删除操作后，继续while循环，调用hasNext方法()判断，由于此时cursor为1，而size为0， 那么返回true，所以继续执行while循环，然后继续调用iterator的next()方法： 

　　注意，此时要注意next()方法中的第一句：checkForComodification()。

　　在checkForComodification方法中进行的操作是：

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
    throw new ConcurrentModificationException();
}
```

如果modCount不等于expectedModCount，则抛出ConcurrentModificationException异常。

　　很显然，此时modCount为1，而expectedModCount为0，因此程序就抛出了ConcurrentModificationException异常。

到这里，想必大家应该明白为何上述代码会抛出ConcurrentModificationException异常了。

<font color='red'>**关键点就在于：调用list.remove()方法导致modCount和expectedModCount的值不一致。**

**注意，像使用for-each进行迭代实际上也会出现这种问题**。</font>

#### 1.6.2、单线程环境下的解决的办法

既然知道原因了，那么如何解决呢？

其实很简单，细心的朋友可能发现在Itr类中也给出了一个remove()方法：

```java
public void remove() {
    if (lastRet == -1)
    throw new IllegalStateException();
       checkForComodification();
 
    try {
    AbstractList.this.remove(lastRet);
    if (lastRet < cursor)
        cursor--;
    lastRet = -1;
    expectedModCount = modCount;
    } catch (IndexOutOfBoundsException e) {
    throw new ConcurrentModificationException();
    }
}
```

在这个方法中，删除元素实际上调用的就是list.remove()方法，但是它多了一个操作： 

`expectedModCount = modCount;` 

因此，在迭代器中如果要删除元素的话，需要调用Itr类的remove方法。

将上述代码改为下面这样就不会报错了：

```java
public class Test {
    public static void main(String[] args)  {
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(2);
        Iterator<Integer> iterator = list.iterator();
        while(iterator.hasNext()){
            Integer integer = iterator.next();
            if(integer==2)
                iterator.remove();   //注意这个地方
        }
    }
}
```

####1.6.2、多线程环境下的解决的办法

面的解决办法在单线程环境下适用，但是在多线程下适用吗？看下面一个例子： 

```java
public class Test {
    static ArrayList<Integer> list = new ArrayList<Integer>();
    public static void main(String[] args)  {
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        Thread thread1 = new Thread(){
            public void run() {
                Iterator<Integer> iterator = list.iterator();
                while(iterator.hasNext()){
                    Integer integer = iterator.next();
                    System.out.println(integer);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
        };
        Thread thread2 = new Thread(){
            public void run() {
                Iterator<Integer> iterator = list.iterator();
                while(iterator.hasNext()){
                    Integer integer = iterator.next();
                    if(integer==2)
                        iterator.remove(); 
                }
            };
        };
        thread1.start();
        thread2.start();
    }
}
```

有可能有朋友说ArrayList是非线程安全的容器，换成Vector就没问题了，实际上换成Vector还是会出现这种错误。 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180624/H95dc9K0d7.png?imageslim)

　原因在于，虽然Vector的方法采用了synchronized进行了同步，**但是实际上通过Iterator访问的情况下，每个线程里面返回的是不同的iterator，也即是说expectedModCount是每个线程私有。**假若此时有2个线程，线程1在进行遍历，线程2在进行修改，那么很有可能导致线程2修改后导致Vector中的modCount自增了，线程2的expectedModCount也自增了，但是线程1的expectedModCount没有自增，此时线程1遍历时就会出现expectedModCount不等于modCount的情况了。 

因此一般有2种解决办法：

　　1）在使用iterator迭代的时候使用synchronized或者Lock进行同步；

　　2）使用并发容器CopyOnWriteArrayList代替ArrayList和Vector。

如果向对多线程并发容器的了解的请自行进行查阅！


## 后言

看了源码才发现源码真的理解需要深厚的基础知识的，弄很多东西都顺手拈来，也是看了源码才知道一个类有哪些地方需要注意，甚至是有哪些地方可以优化，而不是一个API工程师 

## 参考文献

>[Cloneable接口和Object的clone()方法](https://www.cnblogs.com/xrq730/p/4858937.html)
>[RandomAccess 这个空架子有何用？](https://juejin.im/post/5a26134af265da43085de060)
>[Java 集合系列1、细思极恐之ArrayList](https://juejin.im/post/5aec1863518825671c0e6c75)
>[Java ConcurrentModificationException异常原因和解决方法 ](https://www.cnblogs.com/dolphin0520/p/3933551.html)



 

 

 

 

