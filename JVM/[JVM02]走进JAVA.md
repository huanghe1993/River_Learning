# [JVM02]走进JAVA

## java发展史

1991年Jams Gosling 博士领导的绿色计划，此计划是开发一种能够在各种消费性电子产品上运行的程序架构Oak

1995年5月，Oak改名为Java并且在SunWorld大会上提出“Write Once，Run  AnyWhere”

1996年1月，JDK发布1.0 ，JDK1.0提供了一个纯解释型的Java虚拟机（Sun Classic VM）

1997年2月，JDK1.1新增了内部类，反射，jar，jdbc，JavaBeans

1998年12月，JDK1.2  J2SE , j2EE ,Java ME（Nokia s60）  swing新增   出现了Hotspot VM

2000年5月 ,JDK1.3   TImer

2002年2月  JDK1.4，Struts,Hibernate,Spring 1.X  ，正则表达式，NIO,日志，XML解析器

**2004年9月  JDK1.5   语法上很大的改进，自动装箱，拆箱；泛型，注解，枚举，变长参数，增强的for循环  Spring 2.X** 

2006年 12月   JDk1.6   java EE ,java SE ,java ME  提供脚本语言支持（动态语言）提供微型Http服务器Api

2006年11月13日宣布将JDK进行开源

2009年12月 jdk1.7  Lambda项目   （模块化）Jigsaw项目 提出  **Oracle  收购Sun  74亿美元**

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180409/F6dbkf3iji.png?imageslim)

2014年3月19日 JDK1.8正式的发布  Lambda表达式 正式出现    函数式编程   日期/时间改进



## java技术体系

- java程序设计语言
- 各个硬件平台上的java虚拟机
- class文件格式
- java API
- 第三方java类库



## JDK8的新特性

**接口的默认方法和静态方法**

java8使用了两个新概念扩展了接口的含义，默认方法和静态方法。默认方法使得开发者可以在不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制哪些实现了该接口的类也同时实现这个新加的方法

**Lambeda表达式和函数式编程**

函数式编程是技术发展的方向，而Lamber是函数式编程最基础的内容，所以在java8中加入Lamber表达式是符合技术发展的方向的。

通过引入Lambeda，最直观的改进的就是不再需要大量的内部类。可读性好，高阶函数引入函数组合的概念

Lambeda的引入，集合操作也发生了很大的变化。比如stream API，把函数式编程和java集合的概念结合起来



## JAVA虚拟机发展

- Sun Classic  VM
- Exact VM
- Hot Spot VM
- KVM
- JRocket
- J9
- AZul vm
- Liquid VM
- Microsoft VM



Sun  Classic VM是第一款商用的虚拟机，是纯解释器执行java代码，如果要使用JIT编译器，就要使用外挂；这就会有一个问题，如果使用外挂那么JIT编译器就会完全的接管系统，解释器就不会再工作。Hotspot内置了JIT编译器

Exact VM  Exact Memory Management 准确式内存管理，编译器和解释器混合工作及两级即使编译器

Hot Spot VM 这款虚拟机的是Longview Technologies公司开发的，1997年sun完成了收购；优势，**热点代码探测技术**

kvm:运行在移动端上，Android,ios出现之前得到了广泛的应用

JRocket：BEA生产的虚拟机(**Oracle收购了BEA)**    **优势在垃圾收集器，MissControl服务套件**（寻找系统环境中内存泄露的功能）,专注于服务端的应用。

J9:IBM生产的，正式的名字是IT4j,由于太过于拗口就简称j9

Dalvik:Google  安卓平台核心组成部分之一，只能称为虚拟机，而不能称为java虚拟机，它没有遵循java虚拟机的规范，不能直接的执行Java的Class文件，使用的是寄存器架构而不是JVM中常见的栈架构。但是它和java有千丝万缕的练习，它执行的dex文件可以通过Class文件转化而来，使用java语法编写程序，可以直接使用大部分的java API

Microsoft VM:微软公司曾今是Java技术的铁杆粉丝支持者（与Sun公司争夺Java控制权，令Java从跨平台技术变成绑定在windows上的技术是微软公司的目的）在java诞生的初期，它的主要应用目的是在浏览器中运行Java Applets 程序，微软公司为了在IE3中支持java Applets应用而开发了自己的java虚拟机，虽然这款虚拟机只有windows版本，确是当时windows下性能最好的java虚拟机。但是好景不长，在1997年10月，sun公司已侵犯商标、不正当竞争等罪名控告微软（微软赔偿高达10亿美元）并承诺终止java虚拟机的开发，逐步的移除java虚拟机的功能。

Azul VM/Liquid VM ：我们平时所说的“高性能java虚拟机”一般是指HotSpot、JRocket、J9这类通用平台上运行的高性能虚拟机。但其实Azul VM/Liquid VM 这类特定硬件平台的专有虚拟机才是“高性能”武器。

Azul  VM是Azul System公司在HotSpot上进行改进的，运行与Azul System公司专有硬件平台，每个Azul  VM实例都可以管理至少数十个CPU和数百GB的内存，提供巨大范围内实现可控的GC时间的垃圾回收器

Liquid VM ，BEA公司开发的Liquid VM 不需要操作系统，或者是本身就实现了一个专有的操作系统



## Taobao VM

王琤      在HotSpot上自己进行深度定制的一款虚拟机，对硬件的依赖性很大，在编译的过程中，只能使用Intel的CPU,损失了兼容性但是提高了性能（在gc的性能上是非常高的），taobao VM使用了crc32指令，在JNI（java native interface）能降低方法调用的开销

目前在国内使用java最强大的应该属于阿里，阿里很多的开源产品，Dubbo ,Druid

