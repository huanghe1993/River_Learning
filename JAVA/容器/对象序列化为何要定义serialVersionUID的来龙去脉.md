# 对象序列化为何要定义serialVersionUID的来龙去脉

在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便长期保存。比如最常见的是Web服务器中的Session对象，当有10万用户并发访问，就有可能出现10万个Session对象，内存可能吃不消，于是Web容器就会把一些seesion先序列化到内存，等要用了，再还原到对象中，说白了，就是能将一个2进制文件变成内存中的对象。在JAVA中，要实现这种机制，只要实现Serializable接口就可以了，先看下面这个简单例子，serialVersionUID稍后引出。我们先定义一个简单的Person类，然后创建这个对象，最后序列化它到一个文件。  

```java
import java.io.Serializable;  
   
public class Person implements Serializable {  
     
    private String name;  
     
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
}  
```

测试代码：

```java
import java.io.FileInputStream;  
import java.io.FileOutputStream;  
import java.io.ObjectInputStream;  
import java.io.ObjectOutputStream;  
   
public class WhySerialversionUID {  
   
public static void main(String[] args) throws Exception {  
   
//这里是把对象序列化到文件         
Person crab = new Person();  
crab.setName("Mr.Crab");  
   
ObjectOutputStream oo = new ObjectOutputStream  
    (new FileOutputStream("crab_file"));  
oo.writeObject(crab);  
oo.close();  
   
//这里是把文件反序列化为对象
//ObjectInputStream oi = new ObjectInputStream  
//    (new FileInputStream("crab_file"));  
//Person crab_back = (Person) oi.readObject();  
//System.out.println("Hi, My name is " + crab_back.getName());  
//oi.close();  
   
    }  
}  
```

运行完后，我们发现有了一个crab_file文件，这个文件就保存这crab对象在内存中的形态。同样，我们把这部分代码注释掉，运行下面那段还原代码，发现，crab_file文件可以被转化为一个对象。  



一切都那么顺利，但是如果在序列化之后，Person这个类发生了改变呢？比如，多了一个成员变量。我们做如下试验，还是先将对象序列化到一个文件中，之后在Person这个类中添加一个成员变量，如下： 

```java
import java.io.Serializable;  
   
public class Person implements Serializable {  
     
    private String name;  
    //添加这么一个成员变量  
    private String address;  
     
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
}  
```

 之后，我们再去运行一下还原，就发现运行出错了，会报如下错误：  Exception in thread “main” java.io.InvalidClassException: Person; local class incompatible: stream classdesc serialVersionUID = 8383901821872620925, local class serialVersionUID = -763618247875550322  意思就是说，文件流中的class和classpath中的class， 也就是修改过后的class 不兼容了，处于安全机制考虑，程序抛出了错误，并且拒绝载入。那么如果我们真的有需求要在序列化后添加一个字段或者方法呢？应该怎么办？ 

那就是自己去指定serialVersionUID。之前，在我们的例子中，我们是没有指定serialVersionUID的，那么java编译器会自动给这个class进行一个摘要算法，类似于指纹算法，只要这个文件多一个空格，得到的UID就会截然不同的，可以保证在这么多类中，这个编号是唯一的。所以，我们添加了一个字段后，由于没有显指定serialVersionUID，编译器又为我们生成了一个UID，当然和前面保存在文件中的那个不会一样了，于是就出现了2个号码不一致的错误。因此，只要我们自己指定了serialVersionUID，就可以在序列化后，去添加一个字段，或者方法，而不会影响到后期的还原，**还原后的对象照样可以使用，而且还多了方法可以用**，呵呵。但是serialVersionUID我们怎么去生成呢？你可以写1，也可以写2，都无所谓，但是最好还是按照摘要算法，生成一个惟一的指纹数字，eclipse可以自动生成的，jdk也自带了这个工具。一般写法类似于  private static final long serialVersionUID = -763618247875550322L; 