# JAVA中的protected



> **注意**：private和protected只能修饰成员，无法修饰类。类要么是public的，要么是包访问权限。

## 1、Protect方法和属性在包外是不能通过对象访问得到的


> 除了在（2）中表述的有关继承方面的区别之外，在某个类中定义的protected 方法和属性（注意是定义的，不是继承而来的，对于继承而来的情况在（2）中有表述）和默认权限方法和属性是一样的。比如，$\textcolor{Magenta}{某类的protected 方法和属性在包外是不能通过该类对象进行访问的}​$（你能在包外访问一个类的默认权限的方法和属性吗？当然不能），这就是为什么在某对象所在的包的以外的任何地方，你不可以通过该类的对象引用来调用它的protected 方法和属性，**哪怕是在该类的子类中也不可以这样做**。在该类包外的子类中能“看到“的只是子类自己继承来的protected 方法和属性，它是不能“看到“它的父类对象的protected方法和属性的。

结构如下所示：
![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180306/h3cEifdkaI.png?imageslim)

其中Father的代码如下：

```java
package cn.huanghe.protect.one;

public class Father {
    protected int i = 0;
    protected void print(){
        System.out.println(i);
    }
}
```

SonC的代码如下：

```java
package cn.huanghe.protect.two;

import cn.huanghe.protect.one.Father;
import cn.huanghe.protect.one.SonA;

/**
 * Created by huanghe on 2018/3/6.
 */
public class SonC extends Father {
  
  	protected a=6;
    @Override
    protected void print() {
        i = 3;
        System.out.println(i);
    }

    public static void main(String[] args) {
        Father f=new Father();
        System.out.println(f.print());
    }
}
```
报错如下所示：提示没有访问的权限
![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180306/7f0cIL0ma2.png?imageslim)

## 2、

> protected 修饰的方法和属性和默认权限方法和属性的区别在于：在包外的子类可以继承protected 方法和属性，而且被继承的protected 方法和属性，在子类中仍然是protected（如果方法没有被override)，$\textcolor{Magenta}{(从SonC可以看出来)}$但是要注意的是，我这里说它们仍然是protected，是从它们可以由该子类的包外的子类继续继承的递归性角度来说的，实际上它们的可见范围和该子类中新定义的protected 方法和属性是有区别的 。不同之处在于在该子类中新定义的protected 方法和属性对该子类所在的包是可见的。$\textcolor{Magenta}{(从SonC中的protected a=6看出来，这个是包可见的)}$而从父类中继承的protected 方法和属性在该子类所在的包中仅仅对该子类是可见的$\textcolor{Magenta}{(从SonC中的继承过来的print方法看出来，仅仅这个类可见的)}$，同时另外它们还享有被继承前的可见范围（即被被继承前的可见范围仍然保持。这让人想起oop中的一个原则，方法和属性被继承后，其可见的范围只能扩大，不能缩小）。比如某类中定义某个protected方法，那么在该类所在的包中是可以访问该类的包外的子类的通过继承得到的该protected方法的（尽管该子类是在包外）。同时不可以在该类（代号A)的包外定义的某个类B中调用类A的子类SA的继承该类得到的该protected 方法（类B可以是A子类也可以不是A子类，类SA可以是在任何一个包中,但是B和SA是不同的两个类）。
>
> **SonA的代码：**
```java
package cn.huanghe.protect.one;

/**
 * Created by huanghe on 2018/3/6.
 */
public class SonA extends Father {
    @Override
    protected void print() {
        i=1;
        System.out.println(i);
    }
}
```

**SonB的代码**

```java
package cn.huanghe.protect.one;

import cn.huanghe.protect.two.SonC;

/**
 * Created by huanghe on 2018/3/6.
 */
public class SonB extends Father {
    @Override
    protected void print() {
        i=2;
        System.out.println(i);
    }

    public static void main(String[] args) {
        SonA a=new SonA();
        System.out.println("a.i: "+a.i);//同一个包下，可以访问受保护的变量
        a.print();//同一个包下，可以访问受保护的方法
        System.out.println("=========");
        SonC c=new SonC();
        System.out.println(c.i);//
//        c.print();
        //这里会报错，说明当此类与父类在同一个包下时，若被访问的子类与父类不在同一包下，则只可以访问其受保护的变量，不能访问其受保护的方法
    }
}
```

**综述之，protected的确切语义是：protected修饰的方法或变量将会被任何位置的子类继承，但是永远只能被最早定义他的父类所在的包的类所见（除了该类以及其子类能看到本身的该protected方法或变量之外。**

