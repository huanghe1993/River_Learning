# Lambda表达式

Lambda表达式取代了匿名类，取消了模板，允许用函数式风格编写代码。这样有时可读性更好，表达更清晰。 将进一步促进并行第三方库的发展，充分利用多核CPU。 鉴于受Java 8发布的影响最大的是Java集合框架（Java Collections framework），所以最好练习流API和lambda表达式，用于对列表（Lists）和集合（Collections）数据进行提取、过滤和排序 

## 一、基本语法

基本语法: **(parameters) -> expression** 或 **(parameters) ->{ statements; }** 

 Java 中的 Lambda 表达式通常使用 (argument) -> (body) 语法书写，例如：
 (arg1, arg2...) -> { body }

(type1 arg1, type2 arg2...) -> { body },以下是一些例子

(int a, int b) -> {  return a + b; }

() -> System.out.println("Hello World");

(String s) -> { System.out.println(s); }
() -> 42

() -> { return 3.1415 };

一个 Lambda 表达式可以有零个或多个参数

参数的类型既可以明确声明，也可以根据上下文来推断。例如：(int a)与(a)效果相同

所有参数需包含在圆括号内，参数之间用逗号相隔。例如：(a, b) 或 (int a, int b) 或 (String a, int b, float c)

空圆括号代表参数集为空。例如：() -> 42

当只有一个参数，且其类型可推导时，圆括号（）可省略。例如：a -> return a*a

Lambda 表达式的主体可包含零条或多条语句

如果 Lambda 表达式的**主体只有一条语句，花括号{}可省略**。匿名函数的返回类型与该主体表达式一致

如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空


下面是Java lambda表达式的简单例子 

```java
// 1. 不需要参数,返回值为 5
() -> 5
 
// 2. 接收一个参数(数字类型),返回其2倍的值
x -> 2 * x
 
// 3. 接受2个参数(数字),并返回他们的差值
(x, y) -> x – y
 
// 4. 接收2个int型整数,返回他们的和
(int x, int y) -> x + y
 
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)
(String s) -> System.out.print(s)
```

## **二、基本的Lambda例子** 

###2.1、遍历List集合

现在,我们已经知道什么是lambda表达式,让我们先从一些基本的例子开始。 在本节中,我们将看到lambda表达式如何影响我们编码的方式。 假设有一个玩家List ,程序员可以使用 for 语句 ("for 循环")来遍历,在Java SE 8中可以转换为另一种形式: 

```java
 @Test
    public void testLambda(){
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
        for(Integer n: list) {
            System.out.println(n);
        }

        //New way:
        List<Integer> list2 = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
        //当只有一个参数，且其类型可推导时，圆括号（）可省略。例如：a -> return a*a
        //如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式一致
        list2.forEach(n -> System.out.println(n));


        //or we can use :: double colon operator in Java 8
        //使用 Java 8 全新的双冒号(::)操作符将一个常规方法转化为 Lambda 表达式：
        list.forEach(System.out::println);

    }
```



### 2.2、用lambda表达式实现Runnable

我开始使用Java 8时，首先做的就是使用lambda表达式替换匿名类，而实现Runnable接口是匿名类的最好示例。看一下Java 8之前的runnable实现方法，需要4行代码，而使用lambda表达式只需要一行代码。我们在这里做了什么呢？那就是用() -> {}代码块替代了整个[匿名类](http://javarevisited.blogspot.sg/2012/12/inner-class-and-nested-static-class-in-java-difference.html)。 

```java
new Thread(new Runnable() {
    @Override
    public void run() {
    System.out.println("Before Java8, too much code for too little to do");
    }
}).start();

//Java 8方式：
new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
```

### 2.3、使用Lambda实现 Comparator

典型的比较器(按照Name进行升序排序)示例

```java
Comparator<Developer> byName = new Comparator<Developer>() {
    @Override
    public int compare(Developer o1, Developer o2) {
        return o1.getName().compareTo(o2.getName());
    }
};
```

等价的Lambda表达式

```java
Comparator<Developer> byName = (Developer o1, Developer o2)->o1.getName().compareTo(o2.getName());
```

集合升序之后输出：

```java
list.sort((Developer o1, Developer o2)->o1.getName().compareTo(o2.getName());
list.forEach((developer) -> System.out.println(developer));
输出还可以使用
list.forEach(System.out::println)
```

### 2.4、使用lambda表达式和函数式接口Predicate

除了在语言层面支持函数式编程风格，Java 8也添加了一个包，叫做 java.util.function。它包含了很多类，用来支持Java的函数式编程。其中一个便是Predicate，使用 java.util.function.Predicate 函数式接口以及lambda表达式，可以向API方法添加逻辑，用更少的代码支持更多的动态行为。下面是Java 8 Predicate 的例子，展示了过滤集合数据的多种常用方法。Predicate接口非常适用于做过滤。 

```java
public static void main(args[]){
    List languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
 
    System.out.println("Languages which starts with J :");
    filter(languages, (str)->str.startsWith("J"));
 
    System.out.println("Languages which ends with a ");
    filter(languages, (str)->str.endsWith("a"));
 
    System.out.println("Print all languages :");
    filter(languages, (str)->true);
 
    System.out.println("Print no language : ");
    filter(languages, (str)->false);
 
    System.out.println("Print language whose length greater than 4:");
    filter(languages, (str)->str.length() > 4);
}
 
public static void filter(List names, Predicate condition) {
    for(String name: names)  {
        if(condition.test(name)) {
            System.out.println(name + " ");
        }
    }
}
```

输出

```java
Languages which starts with J :
Java
Languages which ends with a
Java
Scala
Print all languages :
Java
Scala
C++
Haskell
Lisp
Print no language :
Print language whose length greater than 4:
Scala
Haskell

```

### 2.5、Java 8中使用lambda表达式的Map和Reduce示例

本例介绍最广为人知的函数式编程概念map。它允许你将对象进行转换。例如在本例中，我们将 costBeforeTax 列表的每个元素转换成为税后的值。我们将 x -> x*x lambda表达式传到 map() 方法，后者将其应用到流中的每一个元素。然后用 forEach() 将列表元素打印出来。使用流API的收集器类，可以得到所有含税的开销。有 toList() 这样的方法将 map 或任何其他操作的结果合并起来。由于收集器在流上做终端操作，因此之后便不能重用流了。你甚至可以用流API的 reduce() 方法将所有数字合成一个，下一个例子将会讲到。 

```java
// 不使用lambda表达式为每个订单加上12%的税
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    System.out.println(price);
}
 
// 使用lambda表达式
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
costBeforeTax.stream().map((cost) -> cost + .12*cost).forEach(System.out::println);

```

在上个例子中，可以看到map将集合类（例如列表）元素进行转换的。还有一个 reduce() 函数可以将所有值合并成一个。Map和Reduce操作是函数式编程的核心操作，因为其功能，reduce 又被称为折叠操作。另外，reduce 并不是一个新的操作，你有可能已经在使用它。SQL中类似 sum()、avg() 或者 count() 的聚集函数，实际上就是 reduce 操作，因为它们接收多个值并返回一个值。流API定义的 reduceh() 函数可以接受lambda表达式，并对所有值进行合并。IntStream这样的类有类似 average()、count()、sum() 的内建方法来做 reduce 操作，也有mapToLong()、mapToDouble() 方法来做转换。这并不会限制你，你可以用内建方法，也可以自己定义。在这个Java 8的Map Reduce示例里，我们首先对所有价格应用 12% 的VAT，然后用 reduce() 方法计算总和。 

```java
// 为每个订单加上12%的税
// 老方法：
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double total = 0;
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    total = total + price;
}
System.out.println("Total : " + total);
 
// 新方法：
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double bill = costBeforeTax.stream().map((cost) -> cost + .12*cost).reduce((sum, cost) -> sum + cost).get();
System.out.println("Total : " + bill);
```

