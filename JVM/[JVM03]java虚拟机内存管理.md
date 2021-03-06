# java虚拟机内存管理

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180410/lFjC9hBI22.png?imageslim)

- 方法区

方法区（Method Area）是各个线程共享的内存区域，它用于存储已被虚拟机加载的**类信息、常量、静态变量**、**即时编译器编译后的代码等数据**。

- Java堆

Java堆（Java Heap）是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建，几乎所有的**对象实例和数组**都在这里分配。Java堆是垃圾收集器管理的主要区域，因此很多时候也称做“GC堆”

- Java虚拟机栈

Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型；**每个方法在执行的同时都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息**

- 本地方法栈

本地方法栈是为虚拟机执行Native方法服务

- 程序计数器

程序计数器（Program Counter Register）是一块较小的内存区域，可以把它看作是当前线程所执行的字节码的行号的指示器

## 程序计数器

- 程序计数器（Program Counter Register）是一块较小的内存区域，可以把它看作是当前线程所执行的字节码的行号的指示器
- 程序计数器是线程私有的，它的生命周期与线程相同
- 如果线程正在执行的是一个Java方法，计数器记录的是正在执行的**虚拟机字节码指令的地址**；如果正在执行的是Native方法，计数器值则为空。
- 此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemeryError情况的区域。

## Java虚拟机栈

- 虚拟机栈描述的是Java方法执行的动态内存模型
- 栈帧
  - 每个方法在执行的同时都会创建一个栈帧（Stack Frame），伴随着方法从创建到执行完成，**就对应着一个栈帧在虚拟机栈中入栈到出栈的过程**。用于存储局部变量表、操作数栈、动态链接、方法出口等信息。
- 存储局部变量表
  - **存放编译时期**可知的各种<font color="red">**基本数据类型，引用数据类型，returnAddress类型**</font>
  - 存储局部变量表所需的内存空间**在编译期就完成了分配**，当进入到方法，需要在帧中分配多少内存是固定的，在方法的运行期间是不会改变局部变量大小。
- 大小
  - 如果虚拟机可以动态扩展（大部分Java虚拟机都可动态扩展，同时也允许固定长度的虚拟机栈），如果扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。
  - 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常

## 本地方法栈

本地方法栈是为虚拟机执行Native方法服务。在虚拟机规范中对本地方法栈使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。

Sun HotSpot虚拟机将本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈也会抛出StackOverflowError和OutOfMemoryError异常。

虚拟机栈和本地方法栈的区别？

> 虚拟机栈是为虚拟机执行java方法服务的，本地方法栈是为虚拟了执行Native方法服务的



## Java堆

- 是Java虚拟机所管理的内存中最大的一块
- **存放对象实例**
- Java堆是垃圾收集器管理的主要区域，因此很多时候也称做“GC堆”。
- 新生代，老年代，Eden空间
- 如果在堆中没有内存完成实例分配，并且堆也无法扩展时，将会抛出OutOfMemoryError异常。
- -Xmx  -Xms

## 方法区

- 它用于存储已被虚拟机加载的**类信息、常量、静态变量、即时编译器编译后**的代码等数据
  - 类信息（类版本，字段，方法，接口）
  - 常量  final
  - 静态变量  static
- 方法区和永久代
- **可以选择不实现垃圾收集**。这区域的内存回收目标主要针<font color="red">**对常量池的回收和对类型卸载**</font>
- 当方法区无法满足内存分配需求时，将抛出OutOfMemory异常


### 运行时常量池

- （Runtime Constant Pool）是方法区中的一部分。Class文件中除了有类的版本，字段，方法，接口等描述信息外，还有一项信息是**常量池，用于存放编译时期生成的各种字面量和引用**，这部分内容将在类加载后进入方法区的运行时常量池中存放。



```java
package com.huanghe.test5;

public class Test {

	public static void main(String[] args) {
		
      //字节码常量
		String s1="abc";
		String s2="abc";
		
		System.out.println(s1 == s2);
		
		String s3=new String("abc");
      
      	System.out.println(s1 == s3);
      
        // s3.intern（）运行时常量
        System.out.println(s1 == s3.intern());

	}

}
```

​	这个例子中我们很容易的会觉得打印出来的是false，但是实际的情况是打印出来的是true，和我们平常认识的，分别创建两个不同的对象，放在堆内存中，然后引用指向的地址是不同的，因此打印出来的是false；这样的理解是错误的。

​	任何一个字符串的创建都是会扔到常量池中去的，常量池在方法区中的存储空间，常量池中有一个叫做StringTable,它的数据类型是HashSet，来存放我们实例化的字符串对象，我们知道HashSet有个特点是无序并且不可重复的，因此abc字符串扔进去只放一次，因此s1==s2;如果是用new的话，创建的对象是在堆内存里面的，所以s1 不等于s3。

​	s3.intern()：这个方法会把在堆内存中的数据搬到运行时的常量池中去，所以System.out.println(s1 == s3.intern());为true



##直接内存

直接内存不是虚拟机运行时内存的一部分，该空间划分在虚拟机外。不过由于直接内存的性能比较好，所以有的工作需要使用直接内存来提高性能。

直接内存会受到物理机剩余可用内存、处理器寻址空间的限制。可以通过NIO和NIO.2来申请直接内存。如果虚拟机堆内存分配太大，可能会导致直接内存空间不足而出现运行时异常。