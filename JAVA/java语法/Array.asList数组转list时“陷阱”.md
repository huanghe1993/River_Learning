# Array.asList:数组转list时“陷阱”

大家都知道这个方法是将数组转成list，是JDK中java.util包中Arrays类的静态方法。**大家使用时一定要注意（请看代码和注释，一看就明了了）：**

```java
String s[]={"aa","bb","cc"};  
List<String> sList=Arrays.asList(s);  
for(String str:sList){//能遍历出各个元素  
    System.out.println(str);  
}  
System.out.println(sList.size());//为3  
  
System.out.println("- - - - - - - - - - -");  
  
int i[]={11,22,33};  
List intList=Arrays.asList(i);  //intList中就有一个Integer数组类型的对象，整个数组作为一个元素存进去的  
for(Object o:intList){//就一个元素  
    System.out.println(o.toString());  
}  
  
System.out.println("- - - - - - - - - - -");  
  
Integer ob[]={11,22,33};  
List<Integer> objList=Arrays.asList(ob);  //数组里的每一个元素都是作为list中的一个元素  
for(int a:objList){  
    System.out.println(a);  
}  
  
System.out.println("- - - - - - - - - - -");  
  
//objList.remove(0);//asList()返回的是arrays中私有的终极ArrayList类型，它有set,get，contains方法，但没有增加和删除元素的方法，所以大小固定,会报错  
//objList.add(0);//由于asList返回的list的实现类中无add方法，所以会报错 
```



之所以出现上面出现的问题，看看asList的源码：

```java
public static <T> List<T> asList(T... a) {  
    return new ArrayList<T>(a);  
    } 
//通过引用Arrays.asList(T...a)方法，可以知道括号中需要一个含数据类型的实参
//（T一般就是泛型的意思），而基本数据类型是没有类型的（有点绕，非要用的话可以借助他们的包装类）
//可是为什么不满足类型还能使用嘞？因为数组也是一个类型
```



```java
private final E[] a;  
  
    ArrayList(E[] array) {  
            if (array==null)  
                throw new NullPointerException();  
        a = array;  
    } 
```

>**这个ArrayList不是java.util包下的**，而是java.util.Arrays.ArrayList，显然它是Arrays类自己定义的一个内部类！这个内部类没有实现add()、remove()方法，而是直接使用它的父类AbstractList的相应方法。而AbstractList中的add()和remove()是直接抛出java.lang.UnsupportedOperationException异常的！
>
>Arrays.asList()返回的是一个List (List是一个接口，返回List实际是返回List接口的一个实现)，这个List在底层是有数组实现的，所以size是fixed的。所以，上面的添加代码是不可以的



而且如果想根据数组得到一个新的正常的list,当然可可以循环一个一个添加，也可以有以下2个种方法：

```java
ArrayList<Integer> copyArrays=new ArrayList<Integer>(Arrays.asList(ob));//这样就                                                           是得到一个新的list，可对其进行add,remove了  
copyArrays.add(222);//正常，不会报错  
  
Collections.addAll(new ArrayList<Integer>(5), ob);//或者新建一个空的list,把要转换的 
```

如果你想直接根据基本类型的数组如int[],long[]直接用asList转成list,那么我们可以选择用apache commons-lang工具包里的数组工具类ArrayUtils类的toObject()方法，非常方便，如下

```java
Arrays.asList(ArrayUtils.toObject(i));//上边的代码：int i[]={11,22,33};，达到了我们想要的效果  
```




**其次，如果不指定返回List的类型(即<>部分)的话，Arrays.asList()对其返回List的类型有自己的判断，可以视为它自身的一种优化机制，如下所示：**

《Thinking in Java》

```java
import java.util.*;  
  
class Snow {}  
  
class Powder extends Snow {}  
class Crusty extends Snow {}  
class Slush extends Snow {}  
  
class Light extends Powder {}  
class Heavy extends Powder {}  
  
public class AsListInference   
{  
    public static void main(String[] args)   
    {  
        List<Snow> snow1 = Arrays.asList(new Crusty(), new Slush(), new Powder()); //pass  
  
        //List<Snow> snow2 = Arrays.asList(new Light(), new Heavy()); //error  
      	//snow2添加2个Snow的导出类对象，按理也是可以的，不过由于它们都是Powder，所以Arrays.asList()返回的是一个List<Powder>。可见Arrays.asList()返回的是精确类型的list。
        List<Powder> snow2 = Arrays.asList(new Light(), new Heavy()); //pass  
          
        List<Snow> snow3 = Arrays.asList(new Light(), new Crusty()); //pass  
  
        List<Snow> snow4 = new ArrayList<Snow>();  
        Collections.addAll(snow4, new Light(), new Heavy()); //pass  
  
        List<Snow> snow5 = Arrays.<Snow>asList(new Light(), new Heavy()); //pass  
    }  
}  
```

snow3混合添加，也没有问题。

snow4不用Arrays.asList()，使用Collections.addAll()，就没有snow2中的局限了。

如果一定要Arrays.asList(new Light(), new Heavy())返回List<Snow>而不是List<Powder>，可以用Arrays.<Snow>asList()来强制产生List<Snow>，如snow5。