# JAVA中static关键字的解析

## 1.static关键字的用途

​	在《Java编程思想》P76中作者是这样说的：当创建类时，就是在描述那个类的外观和行为。除非用new创建那个类的对象，否则，实际并未获得任何对象。执行new来创建对象时，数据存储空间才会被分配，其方法才供外界调用。

​	有两种情形用上述的方法是无解的：1、只想为特定域分配单一空间，而不去考虑究竟需要创建多少对象，甚至根本就不创建任何对象。2、希望某个方法不与包含它的类的任何对象关联在一起。也就是说即使没有创建对象也是可以调用这个方法。

​	通过static关键字就可以很好满足这一需求。当声明一个事物是static的时候，就意味着这个域方法不会与包含它的那个类的任何的实例对象相关联。也就是说即使是没有创建对象也是可以调用这个static方法，或者是访问static域的



#### $\textcolor{}{简而言之：方便在没有创建对象的情况下来进行调用（方法/变量）。}$

　### 1.1、static方法

​	static方法一般称作静态方法，由于静态方法不依赖于任何对象就可以进行访问，因此对于静态方法来说，是没有this的，因为它不依附于任何对象，既然都没有对象，就谈不上this了。并且由于这个特性，在静态方法中不能访问类的非静态成员变量和非静态成员方法，因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用。

​	但是要注意的是，虽然在静态方法中不能访问非静态成员方法和非静态成员变量，但是在非静态成员方法中是可以访问静态成员方法/变量的。举个简单的例子：

​	因此，如果说想在不创建对象的情况下调用某个方法，就可以将这个方法设置为static。我们最常见的static方法就是main方法，至于为什么main方法必须是static的，现在就很清楚了。因为程序在执行main方法的时候没有创建任何对象，因此只有通过类名来访问。

#### 	$\textcolor{Magenta}{另外记住，即使没有显示地声明为static，类的构造器实际上也是静态方法}$	

在thinkingJava中有这样的一句话“和其他任何方法一样，static方法可以创建或使用与其类型相同的被命名对象，因此，static方法常常拿来做“牧羊人”的角色，负责看护与其隶属同一类型的实例群。”其实这句话可以这样理解:

#### $\textcolor{Magenta}{如果你的类代表羊，那么创建的若干的该类的对象，就好像建了一个羊群，然而，类中的}$   $\textcolor{Magenta}{static方法就好比牧羊人一样，无论你的羊群中有多少只羊，而牧羊人只有一个。也就是说，}$  $\textcolor{Magenta}{static方法或属性与对象的创建无关，只和类有关。}$

### 1.2、static变量

　　static变量也称作静态变量，静态变量和非静态变量的区别是：静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。	

```java
class StaticTest{
  static int i=47;
}
//现在即使你创建了两个StaticTest的对象，StaticTest也只有一份存储空间，这两个对象共享同一个i.
StaticTest st1=new StaticTest();
StaticTest st2=new StaticTest();
//在这里st1.i和st2.i访问的同一片内存区域，因此他们的值都是47
StaticTest.i++;
```

引用static变量有两种方法。如前例所示，可以通过对象去定位它，如st2.i；也可以通过类名进行直接的引用，而这对象非静态成员是不行的

使用类名是引用Static变量的首选方式，这不仅是因为它强调了变量的Static结构，而且在某些情况下它还为编译器进行优化提供了更好的机会。

### 1.3、static代码块

​	static关键字还有一个比较关键的作用就是 用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。

　　为什么说static块可以用来优化程序性能，是因为它的特性:只会在类加载的时候执行一次。下面看个例子:

```java
class Person{
    private Date birthDate;
     
    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }
     
    boolean isBornBoomer() {
        Date startDate = Date.valueOf("1946");
        Date endDate = Date.valueOf("1964");
        return birthDate.compareTo(startDate)>=0 && birthDate.compareTo(endDate) < 0;
    }
}
```

　isBornBoomer是用来这个人是否是1946-1964年出生的，而每次isBornBoomer被调用的时候，都会生成startDate和birthDate两个对象，造成了空间浪费，如果改成这样效率会更好：

```java
class Person{
    private Date birthDate;
    private static Date startDate,endDate;
    static{
        startDate = Date.valueOf("1946");
        endDate = Date.valueOf("1964");
    }
     
    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }
     
    boolean isBornBoomer() {
        return birthDate.compareTo(startDate)>=0 && birthDate.compareTo(endDate) < 0;
    }
}
```

　因此，很多时候会将一些只需要进行一次的初始化操作都放在static代码块中进行。



## 2、static关键字的误区

### 2.1、static关键字会改变类中成员的访问权限吗？

​	Java中的static关键字不会影响到变量或者方法的作用域。在Java中能够影响到访问权限的只有private、public、protected（包括包访问权限）这几个关键字。看下面的例子就明白了：

```java
public class App 
{
    public static void main( String[] args )
    {
        System.out.println(Person.name);
        System.out.println(Person.age);//这一句报错：'age' has private access in
    }
}

class Person{
    public static String name="huanghe";
    private static  int age=25;
}

```

System.out.println(Person.age);//这一句报错：'age' has private access in;

这说明static关键字并不会改变变量和方法的访问权限。

### 2.2、能通过this访问静态成员变量吗？

​	虽然对于静态方法来说没有this，那么在非静态方法中能够通过this访问静态成员变量吗？先看下面的一个例子，这段代码输出的结果是什么？

```java
public class Main {　　
    static int value = 33;
 
    public static void main(String[] args) throws Exception{
        new Main().printValue();
    }
 
    private void printValue(){
        int value = 3;
        System.out.println(this.value);
    }
}
```

​	这里面主要考察队this和static的理解。this代表什么？this代表当前对象，那么通过new Main()来调用printValue的话，当前对象就是通过new Main()生成的对象。而static变量是被对象所享有的，因此在printValue中的this.value的值毫无疑问是33。在printValue方法内部的value是局部变量，根本不可能与this关联，所以输出结果是33。在这里永远要记住一点：静态成员变量虽然独立于对象，但是不代表不可以通过对象去访问，所有的静态方法和静态变量都可以通过对象访问（只要访问权限足够）;

### 3、static能作用于局部变量么？

在C/C++中static是可以作用域局部变量的，但是在Java中切记：$\textcolor{Magenta}{static是不允许用来修饰局部变量}$。不要问为什么，这是Java语法的规定。**==static块可以出现类中的任何地方（只要不是方法内部，记住，任何方法内部都不行）==**