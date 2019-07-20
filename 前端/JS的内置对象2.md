## JS的内置对象2

## 1.Data

var date1 =new Date();

设定制定时间：（兼容最强）

var date2 =new Date("2016/01/27 12:00:00")

不常用：

var date3 =new Date('Wed Jan 27 2016 12:00:00GMT+0800 (中国标准时间)');

var date4 =new Date(2016, 1, 27);



## 1.2 Date对象的方法

获取日期和时间

getDate()                 获取日1-31

getDay ()                 获取星期0-6（0代表周日）

getMonth ()             获取月0-11（1月从0开始）

getFullYear ()         获取完整年份（浏览器都支持）

getHours ()         获取小时0-23

getMinutes ()         获取分钟0-59

getSeconds ()         获取秒  0-59

getMilliseconds ()    获取毫秒（1s = 1000ms）

getTime ()         返回累计毫秒数(从1970/1/1午夜)

### 1.3返回距离1970/01/01毫秒数

vardate = Date.now();

vardate = +new Date();

vardate = new Date().getTime();

vardate = new Date().valueOf();

返回1970年 1 月 1日午夜与当前日期和时间之间的毫秒数

## 2.String内置对象

```HTML
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    //简单数据类型无法绑定属性和方法
    var str = "abc";

    var strObj = new String("abc");
    strObj.aaa = 111;
    console.log(strObj);
    console.log(typeof strObj);
    console.log(strObj.aaa);


    //调用了数据类型的转换
    str.aaa = 111;
    console.log(str.aaa);
    console.log(str.length);
    console.log(str.indexOf("c"));


</script>
</body>
</html>
```

### 2.1    给索引查字符(charAt/charCodeAt)

1 charAt，获取相应位置字符（参数： 字符位置）

注释：字符串中第一个字符的下标是 0。如果参数 index 不在 0 与 string.length 之间，该方法将返回一个空字符串。

2 charCodeAt获取相应位置字符编码（参数： 字符位置）

charAt()方法和charCodeAt()方法用于选取字符串中某一位置上的单个字符

区别：charCodeAt()方法，它并不返回指定位置上的字符本身，而是返回该字符在Unicode字符集中的编码值。如果该位置没有字符，返回值为NaN.

字符/字符编码 =Str.charAt/charCodeAt(索引值);

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>
    var str = new String("abcd");
    console.log(str);
        for(var i=0;i<str.length;i++){
            console.log(str.charAt(i));
        }
    console.log(str.charAt(0));
    //    console.log(str.charAt(1));
    //    console.log(str.charAt(2));
    //    console.log(str.charAt(3));
</script>
</body>
</html>
```

### 2.2    给字符查索引indexOf/lastIndexOf()

1 indexOf，从前向后索引字符串位置（参数： 索引字符串）

从前面寻找第一个符合元素的位置

2 lastIndexOf，从后向前索引字符串位置（参数：索引字符串）

从后面寻找第一个符合元素的位置

找不到则返回 -1

索引值 = str.indexOf/lastIndexOf(想要查询的字符);

### 2.3   url 编码和解码（了解）

URI (UniformResourceIdentifiers,通用资源标识符)进行编码，以便发送给浏览器。有效的URI中不能包含某些字符，例如空格。而这URI编码方法就可以对URI进行编码，它们用特殊的UTF-8编码替换所有无效的字符，从而让浏览器能够接受和理解。

encodeURIComponent()函数可把字符串作为 URI 组件进行编码

decodeURIComponent()函数可把字符串作为 URI 组件进行解码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    var str = "abcdea";

    //给字符查索引(索引值为0,说明字符串以查询的参数为开头)
    console.log(str.indexOf("c"));
    console.log(str.lastIndexOf("c"));

    console.log(str.indexOf("a"));
    console.log(str.lastIndexOf("a"));


    //了解；数据传递的时候经常需要通过编码后在传递，接收后还需要反编译回来。
    var url = "http://www.itcast.cn?username='aaa'&password='123'";
    console.log(encodeURIComponent(url));
    console.log(decodeURIComponent(encodeURIComponent(url)));


</script>
</body>
</html>
```

### 2.4    字符串的链接

新字符串 = str1.concat(str2)； 链接两个字符串

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    //concat 连接两个字符串返回一个新字符串，并且不会被修改
//    var str1 = "abc";
//    var str2 = "123";
//    var str3 = str1.concat(str2);
//    console.log(str1);
//    console.log(str2);
//    console.log(str3);

    var str = "I love my family!";
    console.log(str);

//    //slice();   跟剧索引值和索引值截取字符串
//    console.log(str.slice(2));//从索引截取到最后
//    console.log(str.slice(2,5));//从索引截,包左不包右
//    console.log(str.slice(-3));//后几个
//    console.log(str.slice(5,2));//空字符串



//    //substr();   跟剧索引值和长度值截取字符串
//    console.log(str.substr(2));//从索引截取到最后
//    console.log(str.substr(2,6));//从索引截,长度个字符串
//    console.log(str.substr(-3));//后几个


//    //substring();   跟剧索引值和索引值截取字符串
//    console.log(str.substring(2));  //从索引截取到最后
//    console.log(str.substring(2,5));//从索引截,长度个字符串
//    console.log(str.substring(-1)); //全部截取
//    console.log(str.substring(5,2));//只能调换

</script>
</body>
</html>
```

### 2.5  字符串的截取

\1. slice，截取字符串（参数：1，截取位置【必须】，2终结点）

​    字符串 = str.[slice]()（索引1，索引2); 两个参数都是索引值。

(1).（2,5） 正常包左不包右。

(2). ( 2 )     从指定的索引位置剪到最后。

(3).（-3）  从倒数第几个剪到最后.

(4).（5,2） 前面的大，后面的小，空。

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    //去除前后空格，trim();
//    var str1 = "   a   b   c   ";
//    console.log(str1);
//    console.log(str1.trim());


    //replace()替换
//    var str2 = "Today is fine day,today is fine day a!!!"
//    console.log(str2);
//    console.log(str2.replace(/today/gi,"tomorrow"));


    //字符串变数组split();  无参，是把字符串作为一个元素添加进数组中。空字符串，分隔字符串中每一个字符，分别添加进入数组中
            //指定字符分隔数组：特殊符号将不会出现在数组的任意一个元素中

    var str3 = "关羽|张飞|刘备";

    console.log(str3);
    console.log(str3.split("|"));

</script>
</body>
</html>
```

 

\2. substr，截取字符串（参数：1，截取位置【必须】，2截取长度）

​    字符串 = str.substr（参数1，参数2);1索引值,2长度。

​          第一个参数为从索引位置取值，第二个参数返回字符长度。

(1).（2,4）   从索引值为2的字符开始，截取4个字符。

(2).（1）     一个值，从指定位置到最后。

(3).（-3）   从倒数第几个剪到最后.

(4). 不包括前大后小的情况。

\3. substring 同slice

字符串 =str.substring（参数1，参数2); 两个参数都是索引值。

  不同1：参数智能调转位置。

  不同2：参数负值，将全部获取字符串。

   （1）.（2,5）    正常包左不包右。

​    (2).  ( 2 )      从指定的索引位置剪到最后。

​    (3). （-3）   获取全部字符串.

​    (4). （5,2）  前面的大，后面的小，不是空。（2,5）

### 2.6   特殊方法简介

trim()     //只能去除字符串前后的空白

replace()  //替换   str.replace(/aaa/gi,“bbb”);

split()    //字符串变数组







