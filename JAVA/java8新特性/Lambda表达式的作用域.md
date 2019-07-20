# Lambda表达式的作用域

在[Lambda表达式](http://blog.csdn.net/sun_promise/article/details/51121205)中访问外层作用域和旧版本的匿名对象中的方式类似。你可以直接访问标记了final的外层局部变量，或者实例的字段以及静态变量。 

[Lambda表达式](http://blog.csdn.net/sun_promise/article/details/51121205)不会从超类（supertype）中继承任何变量名，也不会引入一个新的作用域。Lambda表达式基于词法作用域，也就是说lambda表达式函数体里面的变量和它外部环境的变量具有相同的语义（也包括lambda表达式的形式参数）。此外，this关键字及其引用，在Lambda表达式内部和外部也拥有相同的语义。 



## 1.访问局部变量

**1）可以直接在lambda表达式中访问外层的局部变量** 

```java
final int num = 1;
Converter<Integer, String> s =
        (param) -> String.valueOf(param + num);
 
s.convert(2);     // 3
```

但是和匿名对象不同的是，**lambda表达式的局部变量**（eg:num）**可以不用声明为final** 

```java
int num = 1;
Converter<Integer, String> s =
        (param) -> String.valueOf(param + num);
 
stringConverter.convert(2);     // 3
```

**2）在 Lambda 表达式当中被引用的变量的值不可以被更改。** 

```java
public void repeat(String string, int count) {
        Runnable runnable = () -> {
            for (int i = 0; i < count; i++) {
                string = string + "a";//编译出错
                System.out.println(this.toString());
            }
        };
        new Thread(runnable).start();
}
```

