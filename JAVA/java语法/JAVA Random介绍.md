# JAVA Random介绍

## 1、简介

Random类位于java.util包下，此类的实例用于生成伪随机数流。之所以称之为伪随机，是因为真正意义上的随机数（或者称为随机事件）在某次产生过程中是按照实验过程表现的分布概率随机产生的，其结果是不可预测，不可见的。而计算机中的随机函数是按照一定的算法模拟产生的，其结果是确定的，可见的。我们认为这样产生的数据不是真正意义上的随机数，因而称之为伪随机。



## 2、Random类的使用

### 2.1生成Random对象

Java API中提供了两个构造方法来new一个Random对象。无参构造底层调用的也是有参构造，将System.nanoTime()作为参数传递。即如果使用无参构造，默认的seed值为System.nanoTime()。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180306/J6Gd4c3JK2.png?imageslim)

> 这里简单说明下System.nanoTime()与System.currentTimeMillis()的区别。System.currentTimeMillis()产生一个当前的毫秒，这个毫秒是自1970年1月1日0时起到当前的毫秒数。而System.nanoTime()是从某个不确定的时间起（同一个的虚拟机上起始时间是固定的），到当前的时间差，它精确到纳秒，这个不确定的起始时间可以是未来，如果起始时间是未来，得到的就是个负数。它的主要用途是衡量一个时间段，比如说一段代码所执行的时间。

```Java
Random random = new Random()
Random random2 = new Random(long seed)
```

对于有参构造，需要注意的是，如果seed值相同，不管执行多少次，随机生成的数据是相同的，具体看下例：

```java
@Test
    public void test01(){
        Random random=new Random(10);
        for (int i = 0; i < 3; i++) {
            System.out.println(random.nextInt());
        }
    }
```

生成指定区间的数值

```Java
@Test
    public void Test02(){
        /**
         * random.nextInt(n)
         * 随机生成一个正整数，整数范围[0,n)
         * 如果想生成其他范围的数据，可以在此基础上进行加减
         *
         * 例如：
         * 1. 想生成范围在[0,n]的整数
         *      random.nextInt(n+1)
         * 2. 想生成范围在[m,n]的整数, n > m
         *      random.nextInt(n-m+1) + m
         *      random.nextInt() % (n-m) + m
         * 3. 想生成范围在(m,n)的整数
         *      random.nextInt(n-m+1) + m -1
         *      random.nextInt() % (n-m) + m - 1
         * ...... 主要是依靠简单的加减法
         */
        Random random = new Random();
        System.out.println("nextInt(10)：" + random.nextInt(10)); //随机生成一个整数，整数范围[0,10)
        for (int i = 0; i < 20; i++) {
            //[3,15)
            //这里有坑，需要注意，如果前面用了+号，应该要把计算结果整体用括号括起来，不然它会把+号解释为字符串拼接
            System.out.println("我生成了一个[3,15)区间的数，它是：" + (random.nextInt(12) + 3));
        }
    }
```



JDK1.8对于Random的新的特性：

```Java
/**
     * 测试Random类中 JDK1.8提供的新方法
     * JDK1.8新增了Stream的概念
     * 在Random中，为double, int, long类型分别增加了对应的生成随机数的方法
     * 鉴于每种数据类型方法原理是一样的，所以，这里以int类型举例说明用法
     */
    @Test
    public void test03() {
        Random random = new Random();
        random.ints();  //生成无限个int类型范围内的数据，因为是无限个，这里就不打印了，会卡死的......
        random.ints(10, 100);   //生成无限个[10,100)范围内的数据
       
        /**
         * 这里的toArray 是Stream里提供的方法
         */
        int[] arr = random.ints(10).toArray();  //生成10个int范围类的个数。
        System.out.println(arr.length);
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
       
       
        //生成5个在[10,100)范围内的整数
        random.ints(5, 10, 100).forEach(System.out :: println); //这句话和下面三句话功能相同
        //forEach等价于：
        arr = random.ints(5, 10, 100).toArray();
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
```