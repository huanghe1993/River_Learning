# JAVA中的包以及路径问题

最近在学习java编程思想的时候，看到第六章访问权限控制的时候，再次把java包package的内容学习一下。

一开始对于package的使用我产生了许多疑惑，不仅是理论上的，在实际写代码的尝试中也出现了许多意想不到的错误。之后自己查阅了许多博客以及《Thinking in  JAVA》上的一些讲解，再结合编写代码试验，总算是稍微弄清楚了关于package的使用，在此为自己的理解做一个笔记。



首先看一下《Thinking in  JAVA》中对于包的一些解释：Java中的一个包就是一个类库单元，包内包含有一组类，它们在单一的名称空间之下被组织在了一起。这个名称空间就是包名。可以使用import关键字来导入一个包。例如使用import java.util.*就可以导入名称空间java.util包里面的所有类。所谓导入这个包里面的所有类，就是在import声明这个包名以后，在接下来的程序中可以直接使用该包中的类。



在其他的博客上看到有人这样的定义包：：package是一个为了方便管理组织java文件的目录结构，并防止不同java文件之间发生命名冲突而存在的一个java特性。不同package中的类的名字可以相同，只是在使用时要带上package的名称加以区分。



在使用package的时候，如果java文件中使用了package，那么该java文件必须放在命名与package名称相同的目录下，比如：

```java
package test
public void Test(){
  
}
```

文件的目录结构就是.....test/Test.java



​	在java的编程思想中提到：我们之所以要导入包名，就是要提供一个管理名称空间的机制。我们知道，如果有两个类A类和B类都含有一个具有相同特征标记（参数列表）的方法f()，即便在同一段代码中同时使用这两个方法f()，也不会发生冲突，原因就在于**有两个不同的类名罩在前面作为限定名，所以两个方法即便同名也不回发生冲突**。其中的重点就是package名称的选择

##### $ \textcolor{Magenta}{（也就是上面提到的不同package中的类的名字可以相同，只是在使用时要带上package的名称加以区分。）}$

####$\textcolor{Magenta}{编写.java文件需要注意的地方}$
​	当编写一个Java源代码文件时，此文件通常被称为编译单元。每个编译单元都必须有一个后缀名.java，而在编译单元内==有且仅有一个==public类，否则编译器就不会接受。该public类的名称必须与文件的名称相同（包括大小写，但不包括后缀名.java）。如果在该编译单元之中还有额外的类的话，那么在包之外的世界是无法看见这些类的，因为它们不是public类，而且它们主要用来为主public类提供支持。

​	 如果使用package语句，它必须是.java文件中除注释以外的第一句程序代码。如果在文件的起始处写：

​	当编译一个.java文件（即一个编译单元）时，在.java文件中的每个类都会有一个输出文件，而该输出文件的名称与.java文件中每个类的名称相同，只是多了一个后缀名.class。因此在编译少量.java文件之后，会得到大量的.class文件。每一个.java文件编译以后都会有一个public类，以及任意数量的非public类。**因此每个.java文件都是一个构件**$\textcolor{Magenta}{（这里就是构件）}$，如果希望许许多多的这样的构件从属于同一个群组，就可以在每一个.java文件中使用关键字package。而这个群组就是一个类库。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180308/cF1AFfm1dA.png?imageslim)



```java
package test;  
  
class Test1 {}  
  
//Test2.java  
  
package test;  
  
public class Test2 {  
      public static void main(String[] args) {  
            Test1 t;  
      }  
}
```

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180308/CecDA4Dkhc.png?imageslim)



从上图可以看出一个.java文件，可以有多个.class文件构成，一个.java文件就是一个构件，如果希望许许多多的构件属于一个群组就要使用package关键字，而一个群组就是一个类库；Java中的一个包就是一个类库单元。



​	作为一名程序员，我们应该牢记：package和import关键字允许做的是将单一的全局名称空间分割成各自独立封闭的名称空间，使得无论多少人使用Internet以及Java开始编写类，都不会出现与我们的类名称相冲突的问题，因为我们的类是被封闭在我们自己定义的独立的名称空间里面的，而非在公共的全局名称空间里面。

​	到这里也许你会发现，其实所谓关键字package打包从未将被打包的东西包装成一个单一的文件，并且一个包可以由许多.class文件构成==（比如说上面的test的包是由Test1.class和Test2.class组成的）==，这就存在将两个名称相同的类打进一个包中的可能。为了避免这种情况的发生，一种合乎逻辑的做法就是将特定的所有.class文件都置于一个目录下。也就是说利用操作系统的层次化的文件结构来解决这一问题。这是Java解决混乱问题的一种方式（这里暂且先不讨论JAR包工具）。

​      将所有的文件收入一个子目录还可以解决另外两个问题：一、怎样创建独一无二的名称；二、怎样查找有可能隐藏于目录结构中某处的类。

​       这些任务是通过将.class文件所在的路径位置编码称package的名称来实现的。

​      按照惯例，package名称的第一部分是类的创建者的反顺序的Internet域名。为什么要用Internet域名呢？因为如果你遵照惯例，Internet域名应该是独一无二的，因此你的package名称也将是独一无二的，也就是前面提到的我们自定义的独立封闭的名称空间将是独一无二的，这样就不会出现名称冲突的问题了。当然，如果你没有自己的域名，你就得构造一组不大可能与他人重复的组合（例如你的姓名），来创立独一无二的package名称。如果你打算发布你的Java程序代码，稍微花费些代价去取得一个域名还是很有必要的。

​      另外，如果你的Java程序代码只是在本地计算机上运行，你还可以把package名称分解为你机器上的一个目录。所以当Java程序运行并且需要加载.class文件的时候，它就可以根据package名称确定.class文件在目录上的所处位置。

​      程序在运行的时候具体是如何确定.class文件位置的呢？

​      来看看Java解释器的运行过程吧：首先，找出环境变量CLASSPATH（可以通过操作系统来设置）。CLASSPATH包含一个或多个目录，用作查找.class文件的根目录==。从根目录开始，解释器获取包名称并将每个句点替换成反斜杠，以从CLASSPATH根中产生一个路径==（例如，package fruit.Apple就变成为fruit/Apple或fruit/Apple或其他，这将取决于操作系统）。得到的路径会与CLASSPATH中的各个不同的根目录路径相连接以获得一个完整的目录路径，解释器就在这些目录中查找与你所需要的类名称相同的.class文件。（此外，解释器还会去查找某些涉及Java解释器所在位置的标准目录。）

​      为了理解这一点，以域名Food.net为例。把它的顺序倒过来，并且全部转换为小写，net.food就成了我们创建类的一个独一无二的名称空间。如果我们决定创建一个名为fruit的类库，我们可以将该名称进一步细分，于是得到一个包名如下：

package net.food.fruit;

 现在，这个包名称就可以用作下面Apple这个文件的名称空间保护伞了：

```java
package net.food.fruit;  
      
    public class Apple  
    {  
        public Apple()  
        {  
        System.out.println("net.food.fruit.Apple");  
        }  
    } 
```

这个文件可能被置于计算机系统中的如下目录中：

​    C:/DOC/JavaT/net/food/fruit

​    之所以要放在这个目录下面是因为前面提到的，便于系统通过CLASSPATH环境变量来找到这个文件。沿着此路径往回看就能看到包名net.food.fruit，但是路径的前半部分怎么办呢？交给环境变量CLASSPATH吧，我们可以在计算机中将环境变量CLASSPATH设置如下：

​    CHASSPATH=.;D:/JAVA/LIB;C:/DOC/JavaT

​    CLASSPATH可以包含多个可供选择的查询路径。每个路径都用分号隔开，可以看到，上面这个CLASSPATH环境值的第三个路径就是我们前面文件的根目录。如前所述，Java解释器将首先找到这个根目录C:/DOC/JavaT，然后将其与包名net.food.fruit相连接，连接的时候将包名中的句点转换成斜杠，就得到完整的class文件路径C:/DOC/JavaT/net/food/fruit。

​    需要补充说明的一点，这里CLASSPATH环境变量关照的是package中的class文件，如果关照的是JAR包中的class文件，则会有一点变化，即，必须在CLASSPATH环境变量路径中将JAR文件的实际名称写清楚，而不仅仅是指明JAR包所在位置目录。可以想象，因为JAR包所在目录位置上可能存在很多别的JAR包，而我们需要使用的那个class文件只会存在于其中一个JAR包里面，因此可以这样理解，这里JAR包实际上也充当了一级文件目录的角色，因此要在CLASSPATH环境变量中写清楚JAR包文件名。例如如果Apple文件存在于名为fruit.jar的JAR文件中，则CLASSPATH应写作：

​    CLASSPATH=.;D:/JAVA/LIB;C:/DOC/JavaT/net/food/fruit.jar

​    一旦路径得以正确建立，下面的文件就可以放于任何目录之下：（==不同包之间的访问==）

```java
import net.food.fruit.*;  
  
public class LibTest  
{  
    public static void main(String[] args)  
    {  
        Apple a=new Apple();  
    }  
} 
```

当编译器碰到fruit库的import语句时，就开始在CLASSPATH所指定的目录中查找，查找过程中分别将CLASSPATH中设定的各项根目录与包名转换来的子目录net/food/fruit相连接，在连接后的完整目录中查找已编译的文件（即class文件）找出名称相符者（对Apple而言就是Apple.class）。找到了这个文件即匹配到了Apple类。