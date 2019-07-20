# JavaScript初步1（基本的数据的类型）

[TOC]



![这里写图片描述](http://img.blog.csdn.net/20171109152119153?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 1.(JS)介绍

◆js是一款运行在客户端的网页编程语言。

◆组成部分

​      ★ecmascript   js标准 

​      ★dom        通过js操作网页元素

​      ★bom        通过api操作浏览器

◆特点

​     ★简单易用

​     ★解释执行

​     ★基于对象

​       面向过程

作用

◆表单验证

◆轮播特效

◆开发游戏

1．验证表单（以前的网速慢）

2．页面特效（PC端的网页效果）

3．移动端（移动web和app）

4．异步和服务器交互（AJAX）

5．服务端开发（nodejs）

![这里写图片描述](http://img.blog.csdn.net/20171109203007694?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1、User Interface  用户界面，我们所看到的浏览器

2、Browserengine  浏览器引擎，用来查询和操作渲染引擎

3、Renderingengine用来显示请求的内容，负责解析HTML、CSS

4、Networking   网络，负责发送网络请求

5、JavaScriptInterpreter(解析者)   JavaScript解析器，负责执行JavaScript的代码

6、UI Backend   UI后端，用来绘制类似组合框和弹出窗口

7、Data Persistence(持久化)  数据持久化，数据存储 cookie、HTML5中的sessionStorage



## （2）JS书写的位置

- 内嵌式

![这里写图片描述](http://img.blog.csdn.net/20171109152505164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 外链式

![这里写图片描述](http://img.blog.csdn.net/20171109152536895?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	

	<script type="text/javascript">

	alert("14期霸气");
	alert("14期威武");
	</script>

</head>
<body>
	14期霸气！
</body>
</html>


```

写js代码的时候，分号不能省略

推荐将JS代码写在html结束标签后边

将多个JS文件合成为一个JS文件（加快页面的加载的速度）

## （3）JS执行过程的小得原理

一：html页面中出现<script>标签后，就会让页面暂停等待脚本的解析和执行。无论当前脚本是内嵌式还是外链式，页面的下载和渲染都必须停下来等待脚本的执行完成才能继续，这在页面的生命周期中是必须的。

例如：通过外链式js文件查看加载速度



## （4）输出消息的几种方式

alert()  在页面弹出一个对话框，早期JS调试使用

```javascript
Confirm()  在页面弹出一个对话框, 常配合if判断使用
点击确定返回true
点击取消返回false
```

Confirm()  在页面弹出一个对话框, 常配合if判断使用

console.log()  将信息输入到控制台，用于js调试

```javascript
console.log("huanghe");
console.error("我是错误");
console.warn("我是肩高");
```

prompt() 弹出对话框，用于接收用户输入的信息

```javascript
var num=prompt("输入电话号码");//num用来接收输入的数值
```

 document.write()在页面输出消息，document.write不仅能输出信息，还能输出标签

```javascript
document.write("<strong>huanghe</strong>");
```

转义字符

```javascript
alert("中\"国\"");

\”转双引
\’转单引
\n转换行
\r 转回车

```

## (5)JS注释

快捷键  ctrl+/

单行注释   //

多行注释  /*  */

## （6）JS变量

◆不能以数字或者纯数字开头来定义变量名。

◆不推荐使用中文来定义变量名。

◆不能使用特殊符号或者特殊符号开头(-除外);

◆不推荐使用关键字和保留字来定义变量名

★在JS中严格区分大小写的！！！



–变量的命名遵守驼峰命名法，首字母小写,第二个单词的首字母大写

•规则(必须遵守)

–由字母、数字、下划线、$组成

–区分大小写

–不能是关键字和保留字



•规范(建议遵守)

–变量的名称要有意义

–变量的命名遵守驼峰命名法，首字母小写,第二个单词的首字母大写

例如：userName



```
变量是在计算机中存储数据的一个标识符。 
变量可以在声明的时候赋值，也可以稍后赋值。
例如：
var number = 50;
var name = "李四";
或者
var name; 
name = “张三”;      （内存显示）


可以在一行上定义多个变量
 var name,age,sex;

```



## （7）基本数据类型

### (7.1)数字类型

#### (7.1.2)进制问题

◆Number   数字类型

   包含正数  负数 小数

★十六进制表示法

从0-9，a(A)-f(F)表示数字。以0x开头

```javascript
var n1=0x43;//十六进制表示的数字
```

★八进制表示法

```javascript
var n5=0345;//八进制表示的数字
```

0开头，0-7组成。

#### （7.1.2）浮点精度丢失问题

```javascript
浮点数值的最高精度是 17 位小数，但在进行算术计算时其精确度远远不如整数
 var result = 0.1 + 0.2; // 结果不是 0.3，而是：0.30000000000000004
 console.log(0.07 * 100);
 永远不要测试某个特定的浮点数值(不要判断两个浮点数是否相等)
```

#### （7.1.3）数值范围问题

```javascript
由于内存的限制，ECMAScript 并不能保存世界上所有的数值
最小值：Number.MIN_VALUE，这个值为： 5e-324
最大值：Number.MAX_VALUE，这个值为： 1.7976931348623157e+308
无穷大：Infinity
无穷小：-Infinity
```

#### （7.1.4）NaN问题


```javascript
NaN 非数值（Not a Number）
console.log(“abc”/18);  //结果是NaN
undefine 与任何类型的数值进行计算其结果都是NaN
NaN 与任何值都不相等，包括 NaN 本身
isNaN() :任何不能被转换为数值的值都会导致这个函数返回 true 
isNaN(NaN);// true
isNaN(“blue”); // true
isNaN(123); // false

```

### （7.2）字符数据类型

◆字符串  String

凡是用双引号或者单引号引起的都是字符串。

```javascript
var a="huanghe";
var b='huang';
```

•要想打印"或者'怎么办？

–var str = "hello\"itcast\"";  //打印输出hello "itcast"

| \n   | 换行   |
| ---- | ---- |
| \t   | 制表   |
| \b   | 空格   |
| \r   | 回车   |
| \f   | 进纸   |
| \‘   | 单引号  |
| \''  | 双引号  |

#### （7.2.1）字符串的不可变

```
ECMAScript 中的字符串是不可变的，也就是说，字符串一旦创建，它们的值就不能改变。
要改变某个变量保存的字符串，首先要销毁原来的字符串，然后再用另一个包含新值的字符串填充该变量
例如：
var str = "123";  str = str + "abc";
 
```

![这里写图片描述](http://img.blog.csdn.net/20171109212842398?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果另外的赋值就需要重新的开辟空间；垃圾回收机制会自动的回收“ac”



### (7.3)布尔数值类型

◆布尔数据类型    Boolean

只有2个值一个是true, 一个是false.   实际运算中true=1,false=0

```javascript
var n1=2;
var n2=3;
alert(n1==n2);//false 错误的
alert(n1<n2);//true正确的
```

```javascript
Boolean类型有两个字面量：true和false，并且区分大小写！
虽然Boolean 类型的字面值只有两个，但 ECMAScript 中所有类型的值都有与这两个 Boolean 值等价的值
例如：
var result = Boolean("a");
console.log(result); //结果是true

var result = Boolean(100); 
console.log(result); //结果是true

```

#### （7.3.1）转化成为Boolean数据类型

•任何类型可以转换成Boolean类型，一般使用在流程控制语句后面

–例如：

var message="hello";

if(message){  alert(message+"world")  };

| 数据类型      | 转化成为true       | 转化为false  |
| --------- | -------------- | --------- |
| Boolean   | true           | false     |
| String    | 任何非空的数据类型      | “ ”空的数据类型 |
| Number    | 任何非零数字值（包含无穷大） | 0和NaN     |
| Object    | 任何的对象          | null      |
| Undefined | n/a            | Undefined |



### (7.4)undefined类型和Null

◆undefined    变量未初始化

定义了变量，没有给变量赋值

```javascript
var n1;
//undefined定义了变量，没有给变量赋值，变量在内存中是存在的
alert(n1);
```

◆null  变量未引用  值为空   object

```javascript
var n2=null;//在内存中找不到这个变量
```

null和undefined有最大的相似性。看看null == undefined的结果(true)也就更加能说明这点。但是null ===undefined的结果(false)。不过相似归相似，还是有区别的，就是和数字运算时，10 + null结果为：10；10 + undefined结果为：NaN。

任何数据类型和undefined运算都是NaN;      但是字符串和undefine的运算是String类型的（undefined）

任何值和null运算，null可看做0运算。

## （8）数据类型的转换

### （8.1）转化为字符串的数据类型

任何类型转化为string的三种方法

1. 变量+“ ”  | 或者是   变量+“abc”
2. string(变量);
3. 变量.toString()    (注意null和undefined是没有toString方法的）

### （8.2）转化为数值的数据类型

此转换容易出现NaN,一旦被转换的变量中含有非数字字符，都是容易出现NaN的

```javascript
   -    *   /   
   
Number(str);   不能把Number(12.34ab)转化为12.34,得到的数值是NaN

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
```

### （8.3）转化为布尔的数据类型

```
Boolean()

！！变量
第一个逻辑非操作会基于无论什么操作数返回一个与之相反的布尔值
第二个逻辑非操作则对该布尔值求反
于是就得到了这个值真正对应的布尔值

!!
主要就是这两种方法
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
    //转换成字符串
//    var bool = true;
//    var num = 111;
//    var aaa;
//    var bbb = null;
//
//    console.log(typeof(bool+""));
//    console.log(typeof(num+""));
//    console.log(typeof(aaa+""));
//    console.log((aaa+""));
//    console.log(typeof(bbb+""));
//
//    console.log(typeof(String(bool)));
//    console.log(typeof(num.toString()));


    //转换成数字
//    var str = "11";
//    var bool = true;
//    console.log(typeof (str-0));
//    console.log(typeof (bool-0));
//    console.log(typeof (str*1));
//    console.log(typeof (bool*1));
//    console.log(typeof (str/1));
//    console.log(typeof (bool/1));
//    console.log(typeof typeof (bool/1));  //数据类型是用string定义的
//
//    console.log(typeof Number(str));
//    console.log(typeof Number(bool));
//
//
//    var str2 = "12.34abc";
//    var str3 = "12.34";
//    console.log(parseInt(str2));
//    console.log(parseFloat(str2));
//    console.log(Number(str3));


    //布尔类型转换
    var date = new Date();    //

    console.log(Boolean(0));
    console.log(Boolean(""));
    console.log(Boolean(null));
    console.log(!!1);
    console.log(!!"abc");
    console.log(!!date);



</script>
</body>
</html>
```



## （9）运算符

```javascript
alert(typeof(n3));//和C语言中的typeof是一样的，但是在java语言中是没有的
```

```
1.1  &&和||运算

 &&链接两个boolean类型，有一个是false结果就是false。

链接值不是布尔类型时，按照成布尔类型计算，结果本身不变。（非布尔）

例子：     1 = 2&&1；       0 = 0 && 1；   都是true取后面，都是false取前面。

||链接两个boolean类型，有一个是true结果就是true。

链接值不是布尔类型时，按照成布尔类型计算，结果本身不变。（非布尔）

例子：2= 2||1；1 = 0|| 1；都是true取前面，都是false取后面。
```



## (10)特殊的算术运算符

★一个数字类型和一个字符串相加，得到的是一个字符串。这个是和java是一样的

★一个数字类型和一个数字字符串相减(将数字的字符串转化为数字)，得到的是一个数字类型。

★一个数字类型和一个非数字字符串相减，得到的是NaN,是一个数字类型。Not a number

◆/ 除号

 ★两个数字类型的变量相除，得到的是一个数字类型。

 ★一个数字类型和一个数字字符串相除，得到的是一个数字类型。

 ★一个数字类型和一个非数字字符串相除，得到的是NaN,是一个数字类型。    

★0做为除数的时候，得到结果  Infinity （无限大），是一个数字类型。

◆%  取余数

 

◆优先级  有()先计算()里边的

| 运算符  | 结果                                       |
| ---- | ---------------------------------------- |
| +    | 如果是数字类型的变量相加，那么结果为数字类型    如果是非数字类型的变量相加，结果为字符串类型 |
| -    | 如果是非数字类型的变量相减结果为  NaN                    |
| *    | 同上                                       |
| /    | 同上 ，如果0作为除数，结果为infinity（无穷大）             |
| %    | 获取余数                                     |
| ()   | 优先级  有括号先计算括号里面的值                        |

## （11）数组的使用

### （11.1）数组的创建

```javascript
数组：数据的有序列表，可以存放任意类型的数据，数组的大小可以动态调整。
创建数组的两种方式
方式1，数组字面量
var arr1 = []; //创建一个空数组，数组字面量
var arr2 = [1, 3, 4]; //创建一个包含3个数值的数组，多个数组项以逗号隔开
var arr3 = ["a", "c"]; // 创建一个包含2个字符串的数组
方式2，Array的构造函数
var arr4 = new Array(); // 创建一个空数组
var arr5 = new Array(10); // 创建一个长度为10的数组
var arr6 = new Array("black", "white", "red"); // 创建一个包含3个字符串的数组
```

```javascript
获取数组中的值
var colors = ["black", "white", "red"]; 
console.log(colors[0]);  //获取第一个元素的值
colors["1"] = "blue"; //给第2个元素重新赋值
console.log(colors);
colors["4"] = "yellow"; //设置第5个元素的值，此时数组中有5个元素
console.log(colors);
length属性，获取或设置数组中元素的个数
console.log(colors.length);//获取数组中元素的个数
colors.length = 1; //设置数组中元素的个数
console.log(colors);

```

```javascript
var arr=[65,97,76,13,27,49,58];
var sum=0
var j=0;
var max;
for (var i = 0; i < arr.length; i++) {
	sum+=arr[i];
}
console.log(sum);

var average=sum/(arr.length);
console.log(average);

for (var i = 1; i < arr.length; i++) {
	if (arr[j]<arr[i]) {
		j=i;
		max=arr[j];
	}
}
console.log(j,max);

var arr1=arr.join("|");
console.log(arr1);

for (var i = 0; i < arr.length/2; i++) {
	var temp=arr[i];
	arr[i]=arr[arr.length-1-i];
	arr[arr.length-1-i]=temp;
}
console.log(arr);

for (var m = 0; m < arr.length-1; m++)
{
	for (var n = arr.length-1; n>m; n--) 
	{
		if (arr[n-1]>arr[n]) 
		{
			var temp=arr[n];
			arr[n]=arr[n-1];
			arr[n-1]=temp;
		}
	}
}

console.log(arr)

```

## (12)进制的转化

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    //了解

    //一个参数，取整。 两个参数，进制转换。
    //任意进制转换成十进制。
    var shiNum = parseInt(111,2);
    console.log(shiNum);

    //toString();   无参转换成字符串。    带参进制转换。
    //十进制转换成任意进制
    var num = 10;
    var renyi = num.toString(16);
    console.log(renyi);

</script>
</body>
</html>
```