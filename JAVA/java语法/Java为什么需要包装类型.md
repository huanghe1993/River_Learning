#JAVA为什么需要包装类型
> 在java程序中，new往往将对象存储在堆里，故用new创建一个对象-特别是小的、简单的变量，往往不是很有效。


|  基本类型   |   包装器类型   |
| :-----: | :-------: |
| boolean |  Boolean  |
|  char   | Character |
|   int   |  Integer  |
|  byte   |   Byte    |
|  short  |   Shrot   |

#### $\textcolor{Magenta}{有了基本类型为什么还要有包装类型呢？} $

>我们都知道在Java语言中，new一个对象存储在堆里，我们通过栈中的引用来使用这些对象；但是对于经常用到的一系列类型如int，如果我们用new将其存储在堆里就不是很有效——特别是简单的小的变量。所以就出现了基本类型，同C++一样，Java采用了相似的做法，对于这些类型不是用new关键字来创建，而是直接将变量的值存储在栈中，因此更加高效

#### $\textcolor{Magenta}{有了基本类型为什么还要有包装类型呢？} $

>我们知道Java是一个面相对象的编程语言，基本类型并不具有对象的性质，为了让基本类型也具有对象的特征，就出现了包装类型（**如我们在使用集合类型Collection时就一定要使用包装类型而非基本类型**），它相当于将基本类型“包装起来”，使得它具有了对象的性质，并且为其添加了属性和方法，丰富了基本类型的操作。 
>
>​     另外，当需要往ArrayList，HashMap中放东西时，像int，double这种基本类型是放不进去的，因为容器都是装object的，这是就需要这些基本类型的包装器类了。

#### $\textcolor{Magenta}{两者之间的相互转换} $

**1.int转Integer**

```java
int i = 0;  
Integer ii = new Integer(i);
```

**2.Integer转int**

```java
Integer ii = new Integer(0);  
int i = ii.intValue(); 
```










