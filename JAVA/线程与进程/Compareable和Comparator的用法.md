# Comparable和Comparator的用法

[TOC]



Java 中为我们提供了两种比较机制：Comparable 和 Comparator 

## 一、Comparable 自然排序

Comparable 是排序接口。

若一个类实现了Comparable接口，就意味着“**该类支持排序**”。  即然实现Comparable接口的类支持排序，假设现在存在“实现Comparable接口的类的对象的List列表(或数组)”，则该List列表(或数组)可以通过 Collections.sort（或 Arrays.sort）进行排序。

此外，“实现Comparable接口的类的对象”可以用作“有序映射(如TreeMap)”中的键或“有序集合(TreeSet)”中的元素，而不需要指定比较器

**Comparable 定义**

Comparable 接口仅仅只包括一个函数，它的定义如下

```java
package java.lang;
import java.util.*;

public interface Comparable<T> {
    public int compareTo(T o);
}
```

Comparable 可以让实现它的类的对象进行比较，具体的比较规则是按照 compareTo 方法中的规则进行。这种顺序称为 **自然顺序**。

compareTo 方法的返回值有三种情况：

- e1.compareTo(e2) > 0 即 e1 > e2
- e1.compareTo(e2) = 0 即 e1 = e2
- e1.compareTo(e2) < 0 即 e1 < e2

注意：

> 1.由于 null 不是一个类，也不是一个对象，因此在重写 compareTo 方法时应该注意 e.compareTo(null) 的情况，即使 e.equals(null) 返回 false，compareTo 方法也应该主动抛出一个空指针异常 NullPointerException。
>
> 2.Comparable 实现类重写 compareTo 方法时一般要求 e1.compareTo(e2) == 0 的结果要和 e1.equals(e2) 一致。这样将来使用 SortedSet 等**根据类的自然排序进行排序**的集合容器时可以保证保存的数据的顺序和想象中一致。

## 二、Comparator 定制排序

Comparator 在 java.util 包下，也是一个接口，JDK 1.8 以前只有两个方法：

```java
public interface Comparator<T> {

    public int compare(T lhs, T rhs);

    public boolean equals(Object object);
}
1234567
```

JDK 1.8 以后又新增了很多方法：

Comparator 则是在外部制定排序规则，然后作为排序策略参数传递给某些类，比如 Collections.sort(), Arrays.sort(), 或者一些内部有序的集合（比如 SortedSet，SortedMap 等）。

使用方式主要分三步：

- 创建一个 Comparator 接口的实现类，并赋值给一个对象 
- 在 compare 方法中针对自定义类写排序规则
- 将 Comparator 对象作为参数传递给 排序类的某个方法
- 向排序类中添加 compare 方法中使用的自定义类

```java
  // 1.创建一个实现 Comparator 接口的对象
        Comparator comparator = new Comparator() {
            @Override
            public int compare(Object object1, Object object2) {
                if (object1 instanceof NewBookBean && object2 instanceof NewBookBean){
                    NewBookBean newBookBean = (NewBookBean) object1;
                    NewBookBean newBookBean1 = (NewBookBean) object2;
                    //具体比较方法参照 自然排序的 compareTo 方法，这里只举个栗子
                    return newBookBean.getCount() - newBookBean1.getCount();
                }
                return 0;
            }
        };

        //2.将此对象作为形参传递给 TreeSet 的构造器中
        TreeSet treeSet = new TreeSet(comparator);

        //3.向 TreeSet 中添加 步骤 1 中 compare 方法中设计的类的对象
        treeSet.add(new NewBookBean("A",34));
        treeSet.add(new NewBookBean("S",1));
        treeSet.add( new NewBookBean("V",46));
        treeSet.add( new NewBookBean("Q",26));
```

## 三、两者之间的差别

```java
import java.util.*;
import java.lang.Comparable;

/**
 * @desc "Comparator"和“Comparable”的比较程序。
 *   (01) "Comparable"
 *   它是一个排序接口，只包含一个函数compareTo()。
 *   一个类实现了Comparable接口，就意味着“该类本身支持排序”，它可以直接通过Arrays.sort() 或 Collections.sort()进行排序。
 *   (02) "Comparator"
 *   它是一个比较器接口，包括两个函数：compare() 和 equals()。
 *   一个类实现了Comparator接口，那么它就是一个“比较器”。其它的类，可以根据该比较器去排序。
 *
 *   综上所述：Comparable是内部比较器，而Comparator是外部比较器。
 *   一个类本身实现了Comparable比较器，就意味着它本身支持排序；若它本身没实现Comparable，也可以通过外部比较器Comparator进行排序。
 */
public class CompareComparatorAndComparableTest{

    public static void main(String[] args) {
        // 新建ArrayList(动态数组)
        ArrayList<Person> list = new ArrayList<Person>();
        // 添加对象到ArrayList中
        list.add(new Person("ccc", 20));
        list.add(new Person("AAA", 30));
        list.add(new Person("bbb", 10));
        list.add(new Person("ddd", 40));

        // 打印list的原始序列
        System.out.printf("Original  sort, list:%s\n", list);

        // 对list进行排序
        // 这里会根据“Person实现的Comparable<String>接口”进行排序，即会根据“name”进行排序
        Collections.sort(list);
        System.out.printf("Name      sort, list:%s\n", list);

        // 通过“比较器(AscAgeComparator)”，对list进行排序
        // AscAgeComparator的排序方式是：根据“age”的升序排序
        Collections.sort(list, new AscAgeComparator());
        System.out.printf("Asc(age)  sort, list:%s\n", list);

        // 通过“比较器(DescAgeComparator)”，对list进行排序
        // DescAgeComparator的排序方式是：根据“age”的降序排序
        Collections.sort(list, new DescAgeComparator());
        System.out.printf("Desc(age) sort, list:%s\n", list);

        // 判断两个person是否相等
        testEquals();
    }

    /**
     * @desc 测试两个Person比较是否相等。
     *   由于Person实现了equals()函数：若两person的age、name都相等，则认为这两个person相等。
     *   所以，这里的p1和p2相等。
     *
     *   TODO：若去掉Person中的equals()函数，则p1不等于p2
     */
    private static void testEquals() {
        Person p1 = new Person("eee", 100);
        Person p2 = new Person("eee", 100);
        if (p1.equals(p2)) {
            System.out.printf("%s EQUAL %s\n", p1, p2);
        } else {
            System.out.printf("%s NOT EQUAL %s\n", p1, p2);
        }
    }

    /**
     * @desc Person类。
     *       Person实现了Comparable接口，这意味着Person本身支持排序
     */
    private static class Person implements Comparable<Person>{
        int age;
        String name;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public int getAge() {
            return age;
        }

        public String toString() {
            return name + " - " +age;
        }

        /**
         * 比较两个Person是否相等：若它们的name和age都相等，则认为它们相等
         */
        boolean equals(Person person) {
            if (this.age == person.age && this.name == person.name)
                return true;
            return false;
        }

        /**
         * @desc 实现 “Comparable<String>” 的接口，即重写compareTo<T t>函数。
         *  这里是通过“person的名字”进行比较的
         */
        @Override
        public int compareTo(Person person) {
            return name.compareTo(person.name);
            //return this.name - person.name;
        }
    }

    /**
     * @desc AscAgeComparator比较器
     *       它是“Person的age的升序比较器”
     */
    private static class AscAgeComparator implements Comparator<Person> {
        
        @Override 
        public int compare(Person p1, Person p2) {
            return p1.getAge() - p2.getAge();
        }
    }

    /**
     * @desc DescAgeComparator比较器
     *       它是“Person的age的升序比较器”
     */
    private static class DescAgeComparator implements Comparator<Person> {
        
        @Override 
        public int compare(Person p1, Person p2) {
            return p2.getAge() - p1.getAge();
        }
    }

}
```

运行结果是：

```java
Original  sort, list:[ccc - 20, AAA - 30, bbb - 10, ddd - 40]
Name      sort, list:[AAA - 30, bbb - 10, ccc - 20, ddd - 40]
Asc(age)  sort, list:[bbb - 10, ccc - 20, AAA - 30, ddd - 40]
Desc(age) sort, list:[ddd - 40, AAA - 30, ccc - 20, bbb - 10]
eee - 100 EQUAL eee - 100
```

