# 泛化的class引用

Class引用总是指向Class的对象，它可以制造类的实例，并包含可作用于这些实例的所有方法代码。它还包含该类的静态成员，因此，Class引用表示的就是它所指的对象的确切类型，而该对象便是Class类的一个对象；

泛型语法可以为编译器强制执行额外的类型检查

```java
public class GenericClassReferences {
    public static void main(String[] args) {
        Class intClass=int.class;
        Class<Integer> genericIntClass=Integer.TYPE;
        //普通类的引用不会产生警告
        intClass=double.class;
        //泛化类的引用只能指向其声明的类型，但是普通的类的引用可以被重新赋值为指向任何其他的Class对象
        //genericIntClass=double.class //illegal
        Class<? extends Number> genericNumberClass=int.class;
        //泛型中，X<Integer>并不是X<Number>的子类型，但是X<Integer>是X<? extends Number>的子类型
        //Class<Number> b=int.class;
    }
}
```

​	在Java SE5中class<?>优于平凡的class,？表示的是通配符，即便他们是等价的，平凡的class不会产生编译器警告信息。class<?>的好处就是它表示你并非是碰巧或者是由于疏忽而使用了一个非具体的类引用，你就是选择的是非具体的版本。

​	