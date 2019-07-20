# java泛型

Java 在 1.5 引入了泛型机制，泛型本质是参数化类型，也就是说变量的类型是一个参数，在使用时再指定为具体类型。泛型可以用于类、接口、方法，通过使用泛型可以使代码更简单、安全。然而 Java 中的泛型使用了类型擦除，所以只是伪泛型。这篇文章对泛型的使用以及存在的问题做个总结，主要参考自 《Java 编程思想》。

## 基本用法

### 泛型类

如果有一个类 `Holder` 用于包装一个变量，这个变量的类型可能是任意的，怎么编写 `Holder` 呢？在没有泛型之前可以这样：

```java
public class Holder1 {
    private Object a;

    public Holder1(Object a) {
        this.a = a;
    }

    public void set(Object a) {
        this.a = a;
    }
    public Object get(){
        return a;
    }

    public static void main(String[] args) {
        Holder1 holder1 = new Holder1("not Generic");
        String s = (String) holder1.get();
        holder1.set(1);
        Integer x = (Integer) holder1.get();
    }
 }
```

在 `Holder1` 中，有一个用 `Object` 引用的变量。因为任何类型都可以向上转型为 `Object`，所以这个 `Holder` 可以接受任何类型。在取出的时候 `Holder` 只知道它保存的是一个 `Object` 对象，所以要强制转换为对应的类型。在 `main` 方法中， `holder1` 先是保存了一个字符串，也就是 `String` 对象，接着又变为保存一个 `Integer` 对象(参数 `1` 会自动装箱)。从 `Holder` 中取出变量时强制转换已经比较麻烦，这里还要记住不同的类型，要是转错了就会出现运行时异常。



下面看看 `Holder` 的泛型版本：

```java
public class Holder2<T> {

    private T a;
    public Holder2(T a) {
        this.a = a;
    }

    public T get() {
        return a;
    }

    public void set(T a) {
        this.a = a;
    }

    public static void main(String[] args) {
        Holder2<String> holder2 = new Holder2<>("Generic");
        String s = holder2.get();

        holder2.set("test");
        holder2.set(1);//无法编译   参数 1 不是 String 类型

    }
}
```

在 `Holder2` 中， 变量 `a` 是一个参数化类型 `T`，`T` 只是一个标识，用其它字母也是可以的。创建 `Holder2` 对象的时候，在尖括号中传入了参数 `T` 的类型，那么在这个对象中，所有出现 `T` 的地方相当于都用 `String` 替换了。现在的 `get` 的取出来的不是 `Object` ，而是 `String` 对象，因此不需要类型转换。另外，当调用 `set` 时，只能传入 `String` 类型，否则编译无法通过。这就保证了 `holder2` 中的类型安全，避免由于不小心传入错误的类型。

通过上面的例子可以看出泛使得代码更简便、安全。引入泛型之后，Java 库的一些类，比如常用的容器类也被改写为支持泛型，我们使用的时候都会传入参数类型，如：`ArrayList<Integer> list = ArrayList<>();`。

### 泛型方法

泛型不仅可以针对类，还可以单独使某个方法是泛型的，举个例子：

```java
public class GenericMethod {
    public <K,V> void f(K k,V v) {
        System.out.println(k.getClass().getSimpleName());
        System.out.println(v.getClass().getSimpleName());
    }

    public static void main(String[] args) {
        GenericMethod gm = new GenericMethod();
        gm.f(new Integer(0),new String("generic"));
    }
}

代码输出：
    Integer
    String
```

`GenericMethod` 类本身不是泛型的，创建它的对象的时候不需要传入泛型参数，但是它的方法 `f` 是泛型方法。在返回类型之前是它的参数标识 `<K,V>`，注意这里有两个泛型参数，所以泛型参数可以有多个。

调用泛型方法时可以**不显式传入泛型参数**，上面的调用就没有。这是因为编译器会使用参数类型推断，根据传入的实参的类型 (这里是 `integer` 和 `String`) 推断出 `K` 和 `V` 的类型。

## 类型擦除

### 什么是类型擦除

Java 的泛型使用了类型擦除机制，这个引来了很大的争议，以至于 Java 的泛型功能受到限制，只能说是”伪泛型“。什么叫类型擦除呢？简单的说就是，**泛型中的类型参数只存在于编译期，在运行时，Java 的虚拟机 ( JVM ) 并不知道泛型的存在。）**。先看个例子：

```java
public class ErasedTypeEquivalence {
 
    public static void main(String[] args) {
 
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        System.out.println(c1 == c2);

    }
}

```

上面的代码有两个不同的 `ArrayList`：`ArrayList<Integer>` 和 `ArrayList<String>`。在我们看来它们的参数化类型不同，一个保存整性，一个保存字符串。但是通过比较它们的 `Class` 对象，上面的代码输出是 `true`。这说明在 JVM 看来它们是同一个类。而在 C++、C# 这些支持真泛型的语言中，它们就是不同的类。



`ArrayList<Integer>` 和 `ArrayList<String>` **在编译的时候是完全不同的类型。你无法在写代码时，把一个 String 类型的实例加到 `ArrayList<Integer>` 中**。但是在程序运行时，的的确确会输出true。

这就是 Java 泛型的类型擦除造成的，因为不管是 `ArrayList<Integer>` 还是 `ArrayList<String>`，在编译时都会被编译器擦除成了 `ArrayList`。Java 之所以要避免在创建泛型实例时而创建新的类，从而避免运行时的过度消耗。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180408/7BDgE4H5ma.png?imageslim)



### 泛型实例在编译之后被擦除了，只保留了原始类型。

```java
public class Test4 {  
    public static void main(String[] args) throws IllegalArgumentException, SecurityException, IllegalAccessException, InvocationTargetException, NoSuchMethodException {  
        ArrayList<Integer> arrayList3=new ArrayList<Integer>();  
        arrayList3.add(1);//这样调用add方法只能存储整形，因为泛型类型的实例为Integer  
        arrayList3.getClass().getMethod("add", Object.class).invoke(arrayList3, "asd");  
        for (int i=0;i<arrayList3.size();i++) {  
            System.out.println(arrayList3.get(i));  
        }  
    }  
```

在程序中定义了一个ArrayList泛型类型实例化为Integer的对象，如果直接调用add方法，那么只能存储整形的数据。不过当我们利用反射调用add方法的时候，却可以存储字符串。这说明了Integer泛型实例在编译之后被擦除了，只保留了<font color="red">原始类型</font>。

在上面，提到了原始类型，什么是原始类型？原始类型（raw type）就是擦除去了泛型信息，最后在字节码中的类型变量的真正类型。无论何时定义一个泛型类型，相应的原始类型都会被自动地提供。类型变量被擦除（crased），**并使用其限定类型（无限定的变量用Object）替换**。

```java
class Pair<T> {  
    private T value;  
    public T getValue() {  
        return value;  
    }  
    public void setValue(T  value) {  
        this.value = value;  
    }  
}  
```

Pair<T>的原始类型为：

```java
class Pair {  
    private Object value;  
    public Object getValue() {  
        return value;  
    }  
    public void setValue(Object  value) {  
        this.value = value;  
    }  
}  
```

因为在`Pair<T>`中，T是一个无限定的类型变量，所以用Object替换。其结果就是一个普通的类，如同泛型加入java变成语言之前已经实现的那样。在程序中可以包含不同类型的Pair，如`Pair<String>`或`Pair<Integer>`，但是，擦除类型后它们就成为原始的Pair类型了，原始类型都是Object。

我们也可以明白`ArrayList<Integer>`被擦除类型后，原始类型也变成了Object，所以通过反射我们就可以存储字符串了。

<font color="red">**如果类型变量有限定，那么原始类型就用第一个边界的类型变量来替换。**</font>

比如Pair这样声明

```java
public class Pair<T extends Comparable& Serializable> {  
```

那么原始类型就是Comparable

### 为什么需要进行泛型的擦除

“迁移兼容性”。由于泛型并不是从Java诞生就存在的一个特性，而是等到SE5才被加入的，所以为了兼容之前并未使用泛型的类库和代码，不得不让编译器擦除掉代码中有关于泛型类型信息的部分，这样最后生成出来的代码其实是『泛型无关』的，我们使用别人的代码或者类库时也就不需要关心对方代码是否已经『泛化』，反之亦然。

### 擦除带来的问题

<font color="red">**在泛型代码内部，无法获得任何有关泛型参数类型的信息。**</font>

```java
class HasF {
    public void f() {
        System.out.println("HasF.f()");
    }
}
public class Manipulator<T> {
    private T obj;

    public Manipulator(T obj) {
        this.obj = obj;
    }

    public void manipulate() {
        obj.f(); //无法编译 找不到符号 f()
    }

    public static void main(String[] args) {
        HasF hasF  = new HasF();
        Manipulator<HasF> manipulator = new Manipulator<>(hasF);
        manipulator.manipulate();

    }
}
```

上面的 `Manipulator` 是一个泛型类，内部用一个泛型化的变量 `obj`，在 `manipulate` 方法中，调用了 `obj` 的方法 `f()`，**但是这行代码无法编译。因为类型擦除，编译器不确定 `obj` 是否有 `f()` 方法**。解决这个问题的方法是给 `T` 一个边界:

```java
class Manipulator2<T extends HasF> {
    private T obj;
    public Manipulator2(T x) { obj = x; }
    public void manipulate() { obj.f(); }
}
```

现在 `T` 的类型是 `<T extends HasF>`，这表示 `T` 必须是 `HasF` 或者 `HasF` 的导出类型。这样，调用 `f()` 方法才安全。`HasF` 就是 `T` 的边界，因此通过类型擦除后，所有出现 `T` 的地方都用 `HasF` 替换。这样编译器就知道 `obj` 是有方法 `f()` 的。

#### 代码片段1

```java
List<Integer> list = new ArrayList<Integer>();
Map<Integer, String> map = new HashMap<Integer, String>();
System.out.println(Arrays.toString(list.getClass().getTypeParameters()));
System.out.println(Arrays.toString(map.getClass().getTypeParameters()));

/* Output
[E]
[K, V]
*/
```

关于`getTypeParameters()`的解释：我们期待的是得到泛型参数的类型，但是实际上我们只得到了一堆占位符。

#### 代码片段2

```java
public class Main<T> {

    public T[] makeArray() {
        // error: Type parameter 'T' cannot be instantiated directly
        return new T[5];
    }
}
```

我们无法在泛型内部创建一个`T`类型的数组，原因也和之前一样，`T`仅仅是个占位符，并没有真实的类型信息，实际上，<font color="red">除了`new`表达式之外，`instanceof`操作和转型（会收到警告）</font>在泛型内部都是无法使用的，而造成这个的原因就是之前讲过的编译器对类型信息进行了擦除。

<font color="red">**同时，面对泛型内部形如`T var;`的代码时，记得多念几遍：它只是个Object，它只是个Object……**</font>

### 既然编译的时候会在方法和类中擦除类型信息，那么在返回对象时又是如何知道其具体类型

#### 代码片段3

```java
public class Holder2<T> {
    private T a;
 
    public Holder2(T a) {
        this.a = a;
    }
 
    public T get() {
        return this.a;
    }
 
    public void set(T a) {
        this.a = a;
    }
 
    public static void main(String[] args) {
        Holder2<String> holder2 = new Holder2<>("test");
        String a = holder2.get();
        System.out.println(a);
    }
}
//结果：test
```

<font color="red">虽然有类型擦除的存在，使得编译器在泛型内部其实完全无法知道有关`T`的任何信息，但是编译器可以保证重要的一点：**内部一致性**，也是我们放进去的是什么类型的对象，取出来还是相同类型的对象，这一点让Java的泛型起码还是有用武之地的。</font>

**代码片段3展现就是编译器确保了我们放在`t`上的类型的确是`T`（即便它并不知道有关`T`的任何类型信息）。这种确保其实做了两步工作：**

- `set()`处的类型检验
- `get()`处的类型转换

这两步工作也成为**边界动作**。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180408/HKa371jHi6.png?imageslim)

### 类型擦除的补偿

类型擦除导致泛型丧失了一些功能，任何在运行期需要知道确切类型的代码都无法工作。要解决这样的问题主要的思想是<font color="red">**显示地传递类型标签（你的类型的class对象）**</font>比如下面的例子：

```java
public class Erased<T> {
    private final int SIZE = 100;
    public static void f(Object arg) {
    if(arg instanceof T) {} // Error
    T var = new T(); // Error
    T[] array = new T[SIZE]; // Error
    T[] array = (T)new Object[SIZE]; // Unchecked warning
    }
}
```

偶尔可以绕过这些问题来编程，但是有时必须通过**引入类型标签来对擦除进行补偿**。这意味着你需要**显式地传递你的类型的Class对象**，以便你可以在类型表达式中使用它。

```java
class Building {}
class House extends Building {}

public class ClassTypeCapture<T> {
  Class<T> kind;
  public ClassTypeCapture(Class<T> kind) {
    this.kind = kind;
  }
  public boolean f(Object arg) {
    return kind.isInstance(arg);
  } 
  public static void main(String[] args) {
    ClassTypeCapture<Building> ctt1 = new ClassTypeCapture<Building>(Building.class);
    System.out.println(ctt1.f(new Building()));
    System.out.println(ctt1.f(new House()));
    ClassTypeCapture<House> ctt2 = new ClassTypeCapture<House>(House.class);
    System.out.println(ctt2.f(new Building()));
    System.out.println(ctt2.f(new House()));
  }
} /* Output:
true
true
false
true
*///:~
```

#### 代码片段4

```java
public class Main<T> {

    public T create(Class<T> type) {
        try {
            return type.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void main(String[] args) {
        Main<String> m = new Main<String>();
        String s = m.create(String.class);
    }
}
```

代码片段4展示了一种用类型标签生成新对象的方法，**但是这个办法很脆弱**，因为这种办法要求对应的类型必须有**默认构造函数**，遇到`Integer`类型的时候就失败了，而且这个错误还不能在编译器捕获。

#### 代码片段5

```java
public class Main<T> {

    public T[] create(Class<T> type) {
        return (T[]) Array.newInstance(type, 10);
    }

    public static void main(String[] args) {
        Main<String> m = new Main<String>();
        String[] strings = m.create(String.class);
    }
}
```

代码片段5展示了对泛型数组的擦除补偿，**本质方法还是通过显示地传递类型标签**，通过`Array.newInstance(type, size)`来生成数组，同时也是最为推荐的在泛型内部生成数组的方法。