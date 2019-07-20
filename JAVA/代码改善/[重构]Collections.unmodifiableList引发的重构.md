# [重构]Collections.unmodifiableList引发的重构

在《重构——改善既有代码的设计》一书中，有一种重构手法叫Encapsulate Collection(封装集群)，

`public static <T> List<T> unmodifiableList(List<? extends T> list)`

参数：list--这是一个不可修改视图是要返回的列表中。

返回值：在方法调用返回指定列表的不可修改视图。

```java
public class CollectionsListDemo {
  public static void main(String[] args) {

      List<String> list = new ArrayList<String>();
      list.add("aa");
     list.add("bb"); 
     
      System.out.println("初始化前: "+ list);
     
      //实现 unmodifiable
      List<Character> immutablelist = Collections.unmodifiableList(list);
     
      //修改 list
      immutablelist.add("zz");     
  }}

结果：

初始化前:: [aa, bb]

Exception in thread "main" java.lang.UnsupportedOperationException
```

2、继续追踪，到底什么场景遇到呢？重构是怎么写的的？

在《重构——改善既有代码的设计》一书中，有一种重构手法叫Encapsulate Collection  ，(封装集合)，

 

定义是：让函数返回这个集合的副本，并在这个类中提供添加/移除集合的副本

 

为了演示该重构手法

 

这本书介绍的是：我们经常用集合，Collection，可能是Array，ArrayList，set，vector保存一组实例，这样的类通常也会提供针对该集合的取值与设值函数

 

但是集合的处理方式和其他种类略有不同，取值函数不应该返回集合本身，因为这会让用户得以修改集合的内容而集合拥有者去一无所知，这样会对用户暴露过多的对象的内部结构信息。如果一个取值函数确实要返回多个值，他应该避免用户直接操作对象内保存的集合，并隐藏对象内与用户无关的的数据结构

 此外，不应该为这个集合提供一个设置函数，但是应该提供对象内集合本身添加/删除的函数，这样，集合拥有者（对象）就可以控制集合元素的添加与删除

 如果做到了上边的一点，集合就很好的封装起来，这样就降低了集合拥有者和用户之间的耦合度

 常用到的做法：

 （1）加入为集合添加/移除元素的函数

（2）将保存的集合的字段初始化一个空的集合

3、下面对上边的内容的例子介绍：

 原来我这样写的：

```java
import java.util.List;

public class Student
{
        private String userName ;
       
        private List<String> courses ;
       

        public Student (String userName , List<String> courses)
       {
               this.userName = userName;
               this.courses = courses;
       }

        public String getUserName()
       {
               return userName ;
       }

        public void setUserName(String userName)
       {
               this.userName = userName;
       }

        public List<String> getCourses()
       {
               return courses ;
       }

        public void setCourses(List<String> courses)
       {
               this.courses = courses;
       }
       
       

}
```

重构后，按照上面介绍的原则，这样重构：

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Student
{
        private String userName ;

        private List<String> courses ;

        public Student (String userName , List<String> courses)
       {
               this.userName = userName;
               this.courses = courses;
       }

        public String getUserName()
       {
               return userName ;
       }

        public void setUserName(String userName)
       {
               this.userName = userName;
       }

        public void addCourse(String course)
       {
               courses.add(course);
       }

        public boolean removeCourse(String course)
       {
               return courses .remove(courses );

       }

        public List<String> getCourses()
       {
               return Collections.unmodifiableList( courses);
       }

        public static void main(String[] args)
       {
              List<String> list = new ArrayList<String>();
              list.add( "数学");
              list.add( "语文");
              Student s = new Student("lily" , list);

              List<String> anotherList = s.getCourses();

               /**
               * throws java.lang.UnsupportedOperationException should replace with
               * s.addCourse(String course)
               */
              anotherList.add( "英语");

               // 不会走到这一步，因为上边抛出了异常
              System. out.println("lily's course.length = " + s.getCourses().size());
       }

}
```