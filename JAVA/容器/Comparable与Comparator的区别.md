# Comparable与Comparator的区别 

今天博主在翻阅TreeMap的源码，发现其键必须是实现Comparable或者Comparator的接口时产生了一些兴趣，比如在TreeMap中的put方法分别对Comparable和Comparator接口分别进行处理。那么疑问就来了，Comparable和Comparator接口的区别是什么，Java中为什么会存在两个类似的接口？

Comparable & Comparator 都是用来实现集合中元素的比较、排序的，**只是 Comparable 是在集合内部定义的方法实现的排序，Comparator 是在集合外部实现的排序**，所以，如想实现排序，就需要在集合外定义 Comparator 接口的方法或在集合内实现 Comparable 接口的方法。

Comparable和Comparator接口都是用来比较大小的，首先来看一下Comparable的定义： 

```java
package java.lang;
import java.util.*;
public interface Comparable<T> {
    public int compareTo(T o);
}
```

Comparator的定义如下： 

```java
package java.util;
public interface Comparator<T> {
    int compare(T o1, T o2);
    boolean equals(Object obj);
}
```

Comparable对实现它的每个类的对象进行整体排序。这个接口需要类本身去实现（这句话没看懂？没关系，接下来看个例子就明白了）。若一个类实现了Comparable 接口，实现 Comparable 接口的类的对象的 List 列表 ( 或数组)可以通过 Collections.sort（或 Arrays.sort）进行排序。此外，实现 Comparable 接口的类的对象 可以用作 “有序映射 ( 如 TreeMap)” 中的键或 “有序集合 (TreeSet)” 中的元素，而不需要指定比较器。 举例（类Person1实现了Comparable接口） 

```java
package collections;
 
public class Person1 implements Comparable<Person1>
{
    private int age;
    private String name;
 
    public Person1(String name, int age)
    {
        this.name = name;
        this.age = age;
    }
    @Override
    public int compareTo(Person1 o)
    {
        return this.age-o.age;
    }
    @Override
    public String toString()
    {
        return name+":"+age;
    }
}
```

可以看到Person1实现了Comparable接口中的compareTo方法。实现Comparable接口必须修改自身的类，即在自身类中实现接口中相应的方法。 

```java
Person1 person1 = new Person1("zzh",18);
        Person1 person2 = new Person1("jj",17);
        Person1 person3 = new Person1("qq",19);
 
        List<Person1> list = new ArrayList<>();
        list.add(person1);
        list.add(person2);
        list.add(person3);
 
        System.out.println(list);
        Collections.sort(list);
        System.out.println(list);
```

输出结果： 

```java
[zzh:18, jj:17, qq:19]
[jj:17, zzh:18, qq:19]
```

如果我们的这个类无法修改，譬如String，我们又要兑取进行排序，当然String中已经实现了Comparable接口，如果单纯的用String举例就不太形象。对类自身无法修改这就用到了Comparator这个接口（策略模式）  

**用 Comparator 是策略模式（strategy design pattern），就是不改变对象自身，而用一个策略对象（strategy object）来改变它的行为。** 

比如：你想对整数采用绝对值大小来排序，Integer 是不符合要求的，你不需要去修改 Integer 类（实际上你也不能这么做）去改变它的排序行为，只要使用一个实现了 Comparator 接口的对象来实现控制它的排序就行了。 

```java
// AbsComparator.java     
import   java.util.*;
 
public   class   AbsComparator   implements   Comparator   {     
    public   int   compare(Object   o1,   Object   o2)   {     
      int   v1   =   Math.abs(((Integer)o1).intValue());     
      int   v2   =   Math.abs(((Integer)o2).intValue());     
      return   v1   >   v2   ?   1   :   (v1   ==   v2   ?   0   :   -1);     
    }     
}
```

可以用下面这个类测试 AbsComparator： 

```java
// Test.java     
import   java.util.*;     
 
public   class   Test   {     
    public   static   void   main(String[]   args)   {     
 
      //产生一个20个随机整数的数组（有正有负）     
      Random   rnd   =   new   Random();     
      Integer[]   integers   =   new   Integer[20];     
      for(int   i   =   0;   i   <   integers.length;   i++)     
      integers[i]   =   new   Integer(rnd.nextInt(100)   *   (rnd.nextBoolean()   ?   1   :   -1));     
 
      System.out.println("用Integer内置方法排序：");     
      Arrays.sort(integers);     
      System.out.println(Arrays.asList(integers));     
 
      System.out.println("用AbsComparator排序：");     
      Arrays.sort(integers,   new   AbsComparator());     
      System.out.println(Arrays.asList(integers));     
    }     
}
```

Collections.sort((List<T> list, Comparator<? super T> c)是用来对list排序的。

如果不是调用sort方法，相要直接比较两个对象的大小，如下：

Comparator定义了俩个方法，分别是   int   compare(T   o1,   T   o2)和   boolean   equals(Object   obj)，
用于比较两个Comparator是否相等

**有时在实现Comparator接口时，并没有实现equals方法，可程序并没有报错，原因是实现该接口的类也是Object类的子类，而Object类已经实现了equals方法** 

Comparable接口只提供了   int   compareTo(T   o)方法，也就是说假如我定义了一个Person类，这个类实现了   Comparable接口，那么当我实例化Person类的person1后，我想比较person1和一个现有的Person对象person2的大小时，我就可以这样来调用：person1.comparTo(person2),通过返回值就可以判断了；而此时如果你定义了一个   PersonComparator（实现了Comparator接口）的话，那你就可以这样： 

再譬如博主遇到的真实案例中，需要对String进行排序，且不区分大小写，我们知道String中的排序是字典排序，譬如：A a D排序之后为A D a，这样显然不对，那么该怎么办呢？同上（下面代码中的list是一个String的List集合）： 

```java
Collections.sort(list, new Comparator<String>()
      {
          @Override
          public int compare(String o1, String o2)
          {
              if(o1 == null || o2 == null)
                  return 0;
              return o1.toUpperCase().compareTo(o2.toUpperCase());
          }
      });
```



Comparable 是排序接口；若一个类实现了 Comparable 接口，就意味着 “该类支持排序”。而 Comparator 是比较器；我们若需要控制某个类的次序，可以建立一个 “该类的比较器” 来进行排序。 前者应该比较固定，和一个具体类相绑定，而后者比较灵活，它可以被用于各个需要比较功能的类使用。可以说前者属于 “静态绑定”，而后者可以 “动态绑定”。 我们不难发现：Comparable 相当于 “内部比较器”，而 Comparator 相当于 “外部比较器”。 