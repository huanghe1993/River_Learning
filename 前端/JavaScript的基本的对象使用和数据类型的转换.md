# JavaScript的基本的对象使用和数据类型的转换

[TOC]

## （1）Date()对象

Date对象是用于处理日期和时间的

## （2）Math是用来处理数学函数的

◆Math.ceil()   天花板函数    向上取整

◆Math.floor()  地板函数



## （3）数据类型的转换

### （3.1）数字类型转换为字符串类型

```
 String() 
```

![这里写图片描述](http://img.blog.csdn.net/20171109180702927?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```
变量.toString()
```

![这里写图片描述](http://img.blog.csdn.net/20171109181005032?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### （3.2） 字符串转数字类型

◆Number

![img](file:///C:/Users/huanghe/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

  ★数字类型的字符串，转换之后得到的数字。

  ★非数字字符串，转换之后得到是NaN。

  ★小数类型的字符串，转换之后得到的是原数字。

◆parseInt

![img](file:///C:/Users/huanghe/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

★整数数字类型的字符串，转换之后得到的整数数字。

★数字开头的字符串，转换之后得到的是前边的数字。

★非数字开头的字符串，转换之后得到的是NaN。

★小数类型的字符串，转换之后取整。

◆parseFloat

★整数数字类型的字符串，转换之后得到的整数数字。

★数字开头的字符串，转换之后得到的是前边的数字。

★非数字开头的字符串，转换之后得到的是NaN。

★小数类型的字符串，转换之后得到的是原数字。

### （3.3）转布尔类型

Boolean()

![img](file:///C:/Users/huanghe/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

★数字和字符串转完之后为true。

★undefined、null、0转完之后为false.

## （4）逻辑运算符

逻辑运算只有2个结果，一个为true,一个为false.

◆且&&

★两个表达式为true的时候，结果为true.

◆或||

★只要有一个表达式为true,结果为true.

◆非！

★和表达式相反的结果

## （5）等号运算符

“=”赋值运算符

“==”只判断内容是否相同，不判断数据类型。

“===”不仅判断内容，还判断数据类型是否相同。

！=  只判断内容是否不相同，不判断数据类型。

！==不全等于不仅判断内容是否不相同，还判断数据类型是否不相同

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<script type="text/javascript">
		var n1=2;
		var n2="2";
		var n3=2;
		alert(n1==n2);
		alert(n2===n3);
	</script>
</head>
<body>
	
</body>
</html>
```

## (6)If...else 条件判断

表达式?结果1：结果2；

如果表达式结果为true,执行结果1，如果表达式结果为false,执行结果2.可以理解为if else  的另外一种写法

## (7)代码调试

## (8)数组的创建

```
var ary=new Array();

var ary1=[];

数组的合并使用ary1.concat(ary2);

var ary2=ary1.join("&")//join返回一个字符串
```

## （9）函数

•函数的定义

```javascript
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

//    //第一种定义方法最强大，定义完毕后，在哪里使用都可以，无位置限制。
//    fn1();
//    //后两种定义方法是有局限性的。（使用函数必须在定义函数之后）
//    fn2();
//    fn3();


    //第一种
    function fn1(){
        console.log("我是第一种定义方法！");
    }


    //第二种(匿名函数),一般用在绑定事件的时候
    var fn2 = function (){
        console.log("我是第二种定义方法！");
    }



    //第三种
    var fn3 = new Function("console.log('我是第三种定义方法！')");


//    fn1();
//    fn2();
//    fn3();


</script>
</body>
</html>
```

function函数名 (){

  程序；

}

•函数的调用

   函数名();       //函数在执行调用的时候调用的位置是没有限制的

### (9.1)函数的参数

```
形参
function f(a,b){}  //a,b是形参，占位用，函数定义时形参无值
实参
var x= 5,y=6;
f(x,y); //x,y实参，有具体的值，会把x,y复制一份给函数内部的a和b，函数内部的值是复制的新值，无法修改外部的x,y
JavaScript中的函数相对于其它语言的函数比较灵(特)活(殊)
在其它语言中实参个数必须和形参个数一致，但是JavaScript中没有函数签名的概念，实参个数和形参个数可以不相等

```

### （9.2）javaScript是没有重载的

```
什么是重载：方法的签名相同(函数返回值、函数名称、函数参数)，其它语言(c++、Java、c#)中有方法的重载。
JavaScript中没有方法的重载
function f1(a,b) {
       return a + b;
}
function f1(a,b,c) {
       return a + b + c;
}
var result = f1(5,6); //NaN
三个参数的f1把两个参数的f1覆盖，调用的是三个参数的f1
证明JavaScript中没有重载


 //如果形参个数与实参个数不匹配...（一般情况下，我们不会让形参和实参不匹配）
    //1.相等的话，正常执行。
    //2.实参大于形参，正常执行。（多余的实参，函数不使用）
    //3.实参小于形参，要看你的程序是否报错。（报错，NaN，undefined）
        //未给定实参的形参为undefined;
```

### （9.3）函数的返回值

```
什么是函数的返回值：

函数程序运行后的结果外部需要使用的时候，我们不能直接给与，需要通过return返回。

总结：函数内部，return后面的值就是返回值；

作用：函数执行后剩下结果就是返回值。
	
函数执行完毕，会不会留下点儿什么，取决于有没有返回值

		var  temp   =    函数名()    =   该函数的返回值;
		
 //如果我们想把函数内部的值赋值为外部，必须使用return;
        //如果没有return或者只有return没有值，那么返回值都是undefined。
        //return可以切断函数，也就是return后面的函数不再执行

```

### (9.4)函数变量的问题

```javascript
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>


<script>

    //变量问题：根据作用范围，变量可以分为局部变量和全局变量。

    //局部变量：只有局部能够访问的变量。
        //函数内部用var定义的变量。
    //全局变量：在哪里都能访问到的变量。
        //函数外部或者进入javascript之后立即定义的变量和函数内部不带有var的变量。

   var num3 = 333;

   //函数加载的时候，只加载函数名，不加载函数体。
   function fn(){
       //局部变量
       var num1 = 111;
       //全局变量（成员变量）
       num2 = 222;

       console.log(num3);
   }

   fn();
   // console.log(num1);
   console.log(num2);
   console.log(num3);


//    //块级作用域，js中没有。
//    {
//        var aaa = 1;
//    }


//     //隐式全局变量
//     function fn(){
//         //b和c都是隐式全局变量
//         var a = b = c = 1;
//         //e和f都是隐式全局变量(分号相当于换行)
//         var d = 1;e =2;f=3;
//         //g和i都不是隐式全局变量
//         var g = 1,h= 2,i=3;
//     }

//     fn();
//     console.log(b);
//     console.log(c);
//     console.log(e);
//     console.log(f);
// //    console.log(a);
// //    console.log(h);
// //    console.log(i);



</script>
</body>
</html>
```

### (9.5)JS的预解析功能

```javascript
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>

<script>
    //预解析：js的解析器在页面加载的时候，首先检查页面上的语法错误。把变量声明提升起来。
    //变量值提升变量名，不提升变量值。而用function直接定义的方法是整体提升。
    //1.查看语法错误。
    //2.变量声明提升和函数整体提升(变量声明提升的时候，只提升变量名，不提升变量值)
    //3.函数范围内，照样适用。
    var aaa;
    console.log(aaa);
    aaa = 111;
    fn();

    function fn(bbb){
        //变量声明提升在函数内部照样实用。
        //函数的就近原则。
        var aaa;
        console.log(aaa);
        aaa = 222;
    }

    function fn2(bbb){
        //两个函数中的局部变量不会相互影响。
        console.log(bbb);
    }

</script>

</body>
</html>
```

## (10)高级函数

### （10.1）匿名函数

定义：匿名函数就是没有名字的函数

作用：（1）不需要定义函数名称

​	   （2）书写起来简便

匿名函数调用的三种方法：

```
一、直接调用或者是自调用（function（）{alter(1)})()
二、时间绑定
三、定时器
```

```javascript
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

//    function fn(){
//        console.log(1);
//    }

    //匿名函数。
//    (function (){
//        console.log(1);
//    })


    //调用方法：
    //1.直接调用
    (function (){
        console.log(1);
    })();

    //2.绑定事件
    document.onclick = function () {
        alert(1);
    }

    //3.定时器
    setInterval(function () {
        console.log(444);
    },1000);


</script>
</body>
</html>
```

### (10.2)函数是一种类型

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    //函数也是一种数据类型。但是，归根结底，他还是属于object。

    var str = "abc";
    var num = 111;
    var boo = true;
    var aaa;
    var bbb = null;//对象类型
    var arr = [];//对象类型

    function fn(){
        alert(1);
//        return 111;
    }

    console.log(typeof str);
    console.log(typeof num);
    console.log(typeof boo);
    console.log(typeof aaa);
    console.log(typeof bbb);
    console.log(typeof arr);
    console.log(typeof fn);
    console.log(typeof fn());

</script>
</body>
</html>

string
05-函数是一种类型.html:25 number
05-函数是一种类型.html:26 boolean
05-函数是一种类型.html:27 undefined
05-函数是一种类型.html:28 object
05-函数是一种类型.html:29 object
05-函数是一种类型.html:30 function   但是归根结底还是属于Object
05-函数是一种类型.html:31 undefined
```

### (10.3)回调函数

把函数作为参数一样进行传递和使用

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>




    //执行函数就等于：函数名+();   整个函数+();
//    fn();
//    (function(){})()

    fn(test);
    //回调函数:函数作为参数进行传递和使用。   有点类似于java的多肽，可以增加扩展性
    function fn(demo){
        demo();
//        test();
    }

    function test(){
        console.log("我是被测试的函数！");
    }


</script>
</body>
</html>
```

```
 //什么情况下，使用回调函数？
    //回调函数一般是用于定义一个规则来使用的。
    //规则的传递只能通过函数实现。通过变量无法达成。所以我们需要传递规则的时候必须使用回调函数。
```

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>
    //什么情况下，使用回调函数？
    //回调函数一般是用于定义一个规则来使用的。
    //规则的传递只能通过函数实现。通过变量无法达成。所以我们需要传递规则的时候必须使用回调函数。
    console.log(fn(10,5,test1));
    console.log(fn(10,5,test2));
    console.log(fn(10,5,test3));
    console.log(fn(10,5,test4));


    function fn(num1,num2,demo){
        return demo(num1,num2);
    }

    //定义四个规则：加减乘除
    function test1(a,b){
        return a+b;
    }
    function test2(a,b){
        return a-b;
    }
    function test3(a,b){
        return a*b;
    }
    function test4(a,b){
        return a/b;
    }


</script>
</body>
</html>
```