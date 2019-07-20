# final的内存语义模型

[TOC]

## 引言

在了解final域的内存语义模型的之前，先来看看final的常见的用法

①：被final修饰的变量是不可变的变量，如果变量是基本的数据类型，数据的值不能改变；如果变量是引用数据类型reference，那么引用不能改变，但是引用指向的对象是可以改变的（在jvm中没有那种语法规定对象是不能进行改变的）。

②：被final修饰的方法是不能被重写的方法，private方法默认使用了final

③：final修饰的类是不能被继承的

问题：为什么final修饰的变量是需要在使用前必须进行初始化呢？而普通的成员变量可以不进行初始化呢？

> 如果是普通的成员变量，是实例化对象的时候，首先为对象分配内存空间，然后系统为成员变量初始化为“零”；如果是使用final进行修饰的成员变量如果没有进行初始化，系统加入进行默认的初始化为0，则后面使用的过程中是无法改变该变量的值的；所以使用final修饰的变量的时候必须要进行初始化，如果没有进行初始编译器会报错，final的初始化是在构造方法内进行的。

## 一、写final域的重排序规则

**写final域的重排序规则禁止把final域的写重排序到构造方法之外**

1：把java的内存模型禁止编译器把final域的写重排序到构造方法之外

2：编译器会在final域之后，在构造方法执行完之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外 

```java
public class FinalTest {

    private int i;  //普通变量
    private final int j;  //final域变量

    public FinalTest(){
        i = 1;
        j = 2;
    }

    private FinalTest obj;

    public void write(){
        obj = new FinalTest();
    }

    public void read(){
        FinalTest test = obj; //读对象引用
        int temp1 = test.i;
        int temp2 = test.j;
    }
}

```

这里假设一个线程A执行writer方法，随后另一个线程B执行reader方法。我们通过这两个线程的交互来说明这两个规则。 

write方法只包含一行代码`demo = new Demo();`这行代码包含两个步骤： 

1：构造一个Demo类型的对象

2：把这个对象的引用赋值为demo

假设线程B的**读对象引用**与**读对象的成员**域之间没有重排序，下图是一种可能的执行时序 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180714/FAKFilJ2H0.png?imageslim)

在上图中，写普通域的操作被编译器重排序到了构造函数之外，读线程B错误的读取到了普通变量i初始化之前的值。而写final域的操作被写final域重排序的规则限定在了构造函数之内，读线程B正确的读取到了final变量初始化之后的值。

　　写final域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的final域已经被初始化了，而普通变量不具有这个保证。以上图为例，读线程B看到对象obj的时候，很可能obj对象还没有构造完成（对普通域i的写操作被重排序到构造函数外，此时初始值1还没有写入普通域i）

　　读final域的重排序规则是：在一个线程中，初次读对象的引用与初次读这个对象包含的final域，JMM禁止重排序这两个操作(该规则仅仅针对处理器)。编译器会在读final域的操作前面加一个LoadLoad屏障。

　　初次读对象引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。由于编译器遵守间接依赖关系，因此编译器不会重排序这两个操作。大多数处理器也会遵守间接依赖，也不会重排序这两个操作。但有少数处理器允许对存在间接依赖关系的操作做重排序（比如alpha处理器），这个规则就是专门用来针对这种处理器的。

## 二、读final的重排序规则

**初次读对象引用与初次读该对象包含的final域，不会发生指令重排序**

初次读对象引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。由于编译器遵守间接依赖关系，因此编译器不会重排序这两个操作

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180714/6iB6cffFfB.png?imageslim)

　　在上图中，读对象的普通域操作被处理器重排序到读对象引用之前。在读普通域时，该域还没有被写线程写入，这是一个错误的读取操作，而读final域的重排序规则会把读对象final域的操作“限定”在读对象引用之后，此时该final域已经被A线程初始化过了，这是一个正确的读取操作。

　　读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。在这个示例程序中，如果该引用不为null，那么引用对象的final域一定已经被A线程初始化过了。

## 三、final域为抽象类型

**在构造方法内对一个final引用的对象的成员域的写入，与随后在构造方法外把这个构造对象的引用赋值给另外一个引用变量，这两个操作之间不能重排序**

```java
public class FinalReferenceTest {

    final int[] arrs;//final引用

    static FinalReferenceTest obj;

    public FinalReferenceTest(){
        arrs = new int[1];//1
        arrs[0] = 1;//2
    }

    public static void write0(){//A线程
        obj = new FinalReferenceTest();//3
    }

    public static void write1(){//线程B
        obj.arrs[0] = 2;//4
    }

    public static void reader(){//C线程
        if(obj!=null){//5
            int temp =obj.arrs[0];//6
        }
    }
}
```

　　在上述的例子中，final域是一个引用类型，它引用了一个int类型的数组，对于引用类型，写final域的重排序规则对编译器和处理器增加了一下的约束：在构造函数内对一个final引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

　　本例中假设线程A先执行write0操作，执行完后线程B执行write1操作，执行完后线程C执行reader操作，下图是一种可能的执行时序：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180714/7cDkaEEBaK.png?imageslim)

　上图中：1是对final域的写入，2是对这个final域引用的对象的成员域的写入，3是把被构造的对象的引用赋值给某个引用变量。**这里除了前面提到的1不能和3重排序外，2和3也不能重排序**。

　　JMM可以确保读线程C至少能看到写线程A在构造函数中对final引用对象的成员域的写入。即C至少能看到数组下标0的值为1。而写线程B对数组元素的写入，读线程C可能看得到，也可能看不到。JMM不保证线程B的写入对读线程C可见，因为写线程B和读线程C之间存在数据竞争，此时的执行结果不可预知。

　　如果想要确保读线程C看到写线程B对数组元素的写入，写线程B和读线程C之间需要使用同步原语（lock或volatile）来确保内存可见性。

##四：构造对象在内存中逸出

　　前面我们提到过，写final域的重排序规则可以确保：在引用变量为任意线程可见之前，该引用变量指向的对象的final域已经在构造函数中被正确初始化过了。其实，要得到这个效果，还需要一个保证：在构造函数内部，不能让这个被构造对象的引用为其他线程所见，也就是对象引用不能在构造函数中“逸出”。

```java
public class FinalReferenceEscapeExample {
　　final int i;
　　static FinalReferenceEscapeExample obj;
　　public FinalReferenceEscapeExample () {
　　　　i = 1; // 1写final域
　　　　obj = this; // 2 this引用在此"逸出"
　　}
　　public static void writer() {
　　　　new FinalReferenceEscapeExample ();
　　}
　　public static void reader() {
　　　　if (obj != null) { // 3
　　　　　　int temp = obj.i; // 4
　　　　}
　　}
}
```

假设一个线程A执行writer()方法，另一个线程B执行reader()方法。这里的操作2使得对象还未完成构造前就为线程B可见。即使这里的操作2是构造函数的最后一步，且在程序中操作2排在操作1后面，执行read()方法的线程仍然可能无法看到final域被初始化后的值，因为这里的操作1和操作2之间可能被重排序。实际的执行时序可能如下： 

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180714/HG47kJ7I2g.png?imageslim)

从上图可以看出：在构造函数返回前，被构造对象的引用不能为其他线程可见。因为此时final域可能还没有初始化。。在构造函数返回后，任意线程都将保证能看到final域正确初始化之后的值。 