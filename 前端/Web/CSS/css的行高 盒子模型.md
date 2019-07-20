# CSS初步3的行高 盒子模型

[TOC]



## （1）行高

- 浏览器默认的文字大小为16px;

- 行高：是基线与基线之间的距离

  行高=文字高度+上下边距

  ![这里写图片描述](http://img.blog.csdn.net/20171108215135712?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

  ​

  上边距到下边距之间的距离=基线到基线之间的距离=行高=文字高度+上下边距=18;

- 一行文字行高和父元素高度一致的时候，垂直居中显示;

- ```
  行高举例的话，假如行高设为30px，那么就是文字中心点距上或者距下各为15px，所以行高与标签的高度一样的时候就垂直居中了
  ```

  如果超出标签的高度的话，会以居上的距离为准，那就会偏下


## (2)行高的单位

| 行高单位 | 文字大小 | 值    |
| ---- | ---- | ---- |
| 20px | 20px | 20px |
| 2em  | 20px | 40px |
| 150% | 20px | 30px |
| 2    | 20px | 40px |

总结:单位除了像素以为，行高都是与文字大小乘积

| 行高单位 | 父元素文字大小 | 子元素文字大小 | 行高   |
| ---- | ------- | ------- | ---- |
| 40px | 20px    | 30px    | 40px |
| 2em  | 20px    | 30px    | 40px |
| 150% | 20px    | 30px    | 30px |
| 2    | 20px    | 30px    | 60px |

总结:不带单位时，行高是和子元素文字大小相乘，em和%的行高是和父元素文字大小相乘。行高以像素为单位，就是定义的行高值。

## （3）盒子模型

所谓盒子模型就是把HTML页面中的元素看作是一个矩形的盒子，也就是一个盛装内容的容器。每个矩形都由元素的内容、内边距（padding）、边框（border）和外边距（margin）组成。

![这里写图片描述](http://img.blog.csdn.net/20171109093247828?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20171109093410186?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### （3.1）边框

Border-top-style:  solid   实线

               dotted  点线
               dashed  虚线
Border-top-color   边框颜色
Border-top-width   边框粗细

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.box{
			width: 300px;
			/*简写的方式是h390*/
			height: 390px;
			background: #999;
			border-top-style: solid;/*线形*/
			border-top-color: red;
			border-top-width: 5px;
			border-bottom-style: dotted;
			border-bottom-color: green;
			border-bottom-width: 10px;

		}
	</style>
</head>
<body>
	<div class="box">黄河</div>
</body>
</html>
```

### （3.2）边框属性连写

特点：没有顺序要求，线型为必写项

```
/*边框属性连写*/
border-top:red solid 5px;

/*四个边框值相同的写法*/
border:12px solid red;
```

### (3.3)边框合并

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		table{
			width: 300px;
			height: 500px;
			border:1px solid red;
			/*合并边框*/
			border-collapse: collapse;
		}
		td{
			border:1px solid red;
		}
	</style>
</head>
<body>
	<table cellspacing="0">
		<tr>
			<td></td>
			<td></td>
			<td></td>
		</tr>
		<tr>
			<td></td>
			<td></td>
			<td></td>
		</tr>
		<tr>
			<td></td>
			<td></td>
			<td></td>
		</tr>
	</table>
</body>
</html>
```

### （3.4）边框的小案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>表单控件案例</title>
	<style type="text/css">
		.username{
			border:0 none;
			outline-style: none;
			border: dashed 1px green;
			background-color:#ccc;	
		}
		.username:focus{
			background-color: red;

		}
		.email{
			border: 0 none;
			outline-style: none;
			border-bottom: dashed red 1px; 
		}
		.search{
			border: 0 none;
			outline-style: none;
			border: 1px solid #999;
			background: url("images/search.png") no-repeat right ;
		}
	</style>
</head>
<body>
	<form action="1.java" method="get">
		<label for="username">用户名：</label> <input type="text" class="username" name="username" id="username"><br><br>
		邮  箱：<input type="text" class="email" name=""><br><br>
		搜索一下：<input type="text" class="search" name=""><br><br>
	</form>
</body>
</html>
```

| 用到的技术点  |                                    |
| ------- | ---------------------------------- |
| 轮廓线     | outline-style:none   取消轮廓线         |
| 获取焦点    | :focus  获取鼠标光标状态                   |
| 取消表单边框  | border:0 none;           兼容性好      |
| label标签 | <label  for="ID名">     友好性 获取光标的id |

### （3.5）内边距

◆padding连写

Padding:20px;   上右下左内边距都是20px

Padding: 20px30px;   上下20px   左右30px

Padding:20px  30px  40px;  上内边距为20px  左右内边距为30px   下内边距为40

Padding:  20px 30px   40px  50px;  上20px 右30px  下40px  左 50px

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.box{
			width: 500px;
			height: 300px;
			/*padding-left: 20px;
			padding-right: 30px;
			padding-top: 40px;
			padding-bottom: 30px;*/
			padding: 20px;
			background-color: yellow;
			border: 1px red solid;
		}
	</style>
</head>
<body>
	<div class="box">黄河</div>
</body>
</html>
```

![这里写图片描述](http://img.blog.csdn.net/20171109104609761?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### （3.6）内边距撑大盒子的问题

◆**内边距撑大盒子的问题******

影响盒子宽度的因素

内边距影响盒子的宽度

边框影响盒子的宽度

盒子的宽度=定义的宽度+边框宽度+左右内边距（让盒子居中可以使用text-aline=center）

```html
<!DOCTYPE html>
<html lang="en"> 
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.box{
			width: 180px;
			height: 300px;
			background-color:green; 
			/*text-align: center;*/
			padding-left: 160px;
			padding-right: 160px;
			border: 10px red solid;
		}
	</style>
</head>
<body>
	<div class="box"><img src="images/1.jpg" alt=""></div>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.big{
			width: 300px;
			height: 150px;
			background-color: pink;
			padding: 75px 100px;
		}
		.small{
			width: 300px;
			height: 150px;
			background-color: red;
		}
	</style>
</head>
<body>
	<div class="big"><div class="small"></div></div>
</body>
</html>
```

如果需要盒子在另外的一个盒子居中显示的话，把父盒子的宽度和子盒子的宽度设置成为一样的值，然后再调整父盒子的内边距

◆继承的盒子一般不会被撑大**

继承的盒子的子盒子的宽度是继承负盒子的宽度，但是高度不不会继承的，高度为0；



包含（嵌套）的盒子，如果**子盒子没有定义宽度，给子盒子设置左右内边距，一般不会撑大盒子**，这里是指的是水平的方向上，在垂直的方向上一样的会撑大盒子的宽度。会让子盒子的高度增加；而父盒子的高度不会变大

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .father{
            width: 500px;
            height: 300px;
            background-color: #cccccc;
        }
        .son{
            height: 100px;
            background-color: green;
            /*继承的盒子不会被撑大*/
            padding-left: 10px;
        }
    </style>
</head>
<body>
<div class="father">
    <div class="son"></div>
</div>
</body>
</html>
```

### （3.7）外边距

margin-left   |right  | top  |  bottom

◆外边距连写

Margin:20px;    上下左右外边距20PX

Margin: 20px 30px;   上下20px  左右30px

Margin: 20px  30px  40px;    上20px  左右30px   下  40px

Margin: 20px  30px   40px 50px; 上20px   右30px   下40px  左50px



◆**垂直方向外边距合并**

两个盒子垂直一个设置上外边距，一个设置下外边距，取的设置较大的值。



◆**嵌套的盒子外边距塌陷******

嵌套的盒子，直接给子盒子设置垂直方向外边距的时候，会发生外边距的塌陷

解决方法:  1  给父盒子设置边框

​        	   2给父盒子overflow:hidden;   bfc   格式化上下文

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        /*<!-- 解决方法:  1  给父盒子设置边框*/

        /*2给父盒子overflow:hidden;   bfc   格式化上下文-->*/
        .father{
            width: 500px;
            height: 400px;
            background-color: #cc6600;
            /*不建议使用*/
            /*border: 1px solid #cc6600;*/
            overflow: hidden;
        }
        .son{
            width: 200px;
            height: 200px;
            background-color: #eeeeee;
            margin-top: 30px;
        }
    </style>
</head>
<body>
<div class="father">
    <div class="son"></div>
</div>
</body>
</html>
```

[http://www.w3cplus.com/css/understanding-bfc-and-margin-collapse.html](http://www.w3cplus.com/css/understanding-bfc-and-margin-collapse.html)



 行内元素可以定义左右的内外边距，上下会被忽略掉。

  **行内块可以定义内外边距。**

## （4）文档流布局（标准流）

元素自上而下，自左而右，块元素独占一行，行内元素在一行上显示，碰到父集元素的边框换行

## （5）浮动布局

float: left   |   right

特点：

★元素浮动之后不占据原来的位置,浮动之后脱离了文档流（脱标）

★浮动的盒子在一行上显示

★行内元素浮动之后转换为行内块元素。（不推荐使用，转行内元素最好使用

display: inline-block;）

### （5.1）浮动的作用

◆文本绕图：文字遇到浮动的元素的时候是没有层叠关系的，文字不参与浮动

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .box{
            width: 400px;
            height: 300px;
            background: #eee;
        }
        .box img{
           float: left;
        }
    </style>
</head>
<body>
    <div class="box">
        <img src="../images/shanshan.jpg" alt=""/>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
        <p>当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误</p>
    </div>
</body>
</html>
```

◆制作导航

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        body,ul,li{
            margin: 0;
            padding: 0px;
        }
        ul,li{
            list-style: none;
        }
        .nav{
            width: 800px;
            height: 40px;
            background: pink;
            margin: 20px auto;
        }
        .nav ul li{
            float: left;
        }
        .nav ul li a{
            display: block;
            height: 40px;
            font: 14px/40px 微软雅黑 ;
            padding:0 20px ;
            text-decoration: none;
        }
        .nav ul li a:hover{
            background: #aaa;
        }
    </style>
</head>
<body>
<div class="nav">
    <ul>
        <li><a href="#">百度</a></li>
        <li><a href="#">百度一下</a></li>
        <li><a href="#">黄河</a></li>
    </ul>
</div>
</body>
</html>
```

◆网页布局

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .header,.main,.footer{
            width: 500px;    

        }
        .header,.footer{
            height: 100px;
            background: pink;
        }
        .main{
            height: 300px;
            background: blue;
        }
        .left,.right{
            width: 100px;
            height: 300px;
          
        }
        .left{
            background: orange;
            float: left;
        }
        .content{
            width: 300px;
            height: 300px;
            background: yellow;
            float: left;
        }
        .right{
            background: green;
            float: right;
        }
        .content-top,.content-bottom{
            height: 150px;
        }
        .content-top{
            background: #660000;
        }
        .content-bottom{
            background: blue;
        }
    </style>
</head>
<body>
    <div class="header"></div>
    <div class="main">
        <div class="left"></div>
        <div class="content">
            <div class="content-top"></div>
            <div class="content-bottom"></div>
        </div>

        <div class="right"></div>
    </div>
    <div class="footer"></div>
</body>
</html>
```

### （5.2）清除浮动

当父盒子没有定义高度，嵌套的盒子浮动之后，下边的元素发生位置错误。

◆清除浮动不是不用浮动，清除浮动产生的不利影响。

◆清除浮动的方法

clear: left  | right  | both

工作里用的最多的是clear:both;

★额外标签法

 在最后一个浮动元素后添加标签

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .header,.main,.footer{
            width: 500px;    

        }
        .header{
            height: 100px;
            background: #000;
        }
        .main{
            /*height: 300px;*/
            background: #eeeeee;
            margin: 10px 0;

        }

        .content{
            width: 300px;
            height: 300px;
            background: orange;
            float: left;
        }
        .sidebar{
            width: 190px;
            height: 300px;
            background: green;
            float: right;

        }
        .footer{
            height: 100px;
            background: #000;
        }
    </style>
</head>
<body>
    <div class="header"></div>
    <div class="main">
        <div class="content"></div>
        <div class="sidebar"></div>
     	 <--额外标签法-->
        <div style="clear: both;"></div>
    </div>
    <div class="footer"></div>
</body>
</html>
```



★给父集元素使用overflow:hidden;    bfc 

如果有内容出了盒子，不能使用这个方法

```css
    .main{
            /*height: 300px;*/
            background: #eeeeee;
            margin: 10px 0;
            /*给父集元素添加overflow: hidden*/
            overflow: hidden;

        }
```



★伪元素清除浮动  推荐使用

```css
  .clearfix:after{
            content: ".";
            display: block;
            height: 0px;
            line-height: 0;
            visibility: hidden;
            clear: both;
        }
        /*兼容IE*/
        .clearfix{
            zoom: 1;
        }
```

### （5.3）overflow

![img](file:///C:/Users/huanghe/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

## （6）定位布局

#### (6.1)静态定位

定位方向: left  | right  | top | bottom

◆position:static; 静态定位。默认值，就是文档流



#### （6.2）绝对定位

◆绝对定位

Position:absolute;

特点：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .box{
            width: 100px;
            height: 100px;
            background: red;
            /*静态定位*/
            position: absolute;
        }
        .box1{
            width: 200px;
            height: 100px;
            background:green;
        }
    </style>
</head>
<body>

<div class="box"></div>
<div class="box1"></div>
</body>
</html>
```

★元素使用绝对定位之后不占据原来的位置（脱标）

```css
 .box{
            width: 100px;
            height: 100px;
            background: red;
            /*静态定位*/
            position: absolute;
            bottom: 100px;
            right: 100px ;
```

★元素使用绝对定位，位置是从浏览器出发。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        body{
            margin: 0;
        }
        .father{
            width: 300px;
            height: 300px;
            background: green;
        }
        .box{
            width: 100px;
            height: 100px;
            background: red;
            /*静态定位*/
            position: absolute;
            right: 100px;
        }
    </style>
</head>
<body>

<div class="father">
    <div class="box"></div>
</div>


</body>
</html>
```

★嵌套的盒子，父盒子没有使用定位，子盒子绝对定位，子盒子位置是从浏览器出发。

```css
   <style>
        body{
            margin: 0;
        }
        .father{
            width: 300px;
            height: 300px;
            background: green;
        }
        .box{
            width: 100px;
            height: 100px;
            background: red;
            /*静态定位*/
            position: absolute;
            right: 100px;
            /*bottom: 100px;*/
            /*right: 100px ;*/
        }
```

★嵌套的盒子，父盒子使用定位，子盒子绝对定位，子盒子位置是从父元素位置出发。

```html
        .baby{
            width: 100px;
            height: 100px;
            background:yellow;
            position: absolute;
        }
    </style>
</head>
<body>

<div class="father">
    <div class="box"></div>
</div>
<span class="baby"></span>
```

★给行内元素使用绝对定位之后，转换为行内块。（不推荐使用，推荐使用display:inline-block;）

#### （6.4）相对定位

◆相对定位

Position: relative;

特点： 

★使用相对定位，位置从自身出发。

★还占据原来的位置。

★**子绝父相（父元素相对定位，子元素绝对定位）******最后的子元素的位置是相对于父的

★行内元素使用相对定位不能转行内块

#### （6.5）固定定位 

◆固定定位

Position:fixed;

特点：

★固定定位之后，不占据原来的位置（脱标）

★元素使用固定定位之后，位置从浏览器出发。

★元素使用固定定位之后，会转化为行内块（不推荐，推荐使用display:inline-block;） 

## （7）定位的盒子进行居中对齐

★:margin:0 auto;  只能让标准流的盒子水平居中对齐。

★:text-aline:center是使内容水平居中对齐

★:line-height是使文字垂直居中

★定位的盒子居中：先向右走父元素盒子的一半50%，在向左走子盒子的一半(margin-left:负值。)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .box{
            height: 500px;
            background: #aaa;
            position: relative;
        }
        .nav{
            width: 960px;
            height: 60px;
            background: #666;
            position: absolute;
            bottom: 0px;
            left: 50%;
            margin-left: -480px;
        }
    </style>
</head>
<body>
<div class="box">
    <div class="nav"></div>
</div>
</body>
</html>
```

```css
<style>
        body,ul,li{
            margin: 0;
            padding: 0;
        }
        ul,li{
            list-style: none;
        }
        
    </style>
```

## (8)标签的包含规范

◆div可以包含所有的标签。

◆p标签不能包含div h1等标签。

◆h1可以包含p，div等标签。

◆行内元素尽量包含行内元素，行内元素不要包含块元素<span><a src=""></a></span>

## (9)规避脱标流

◆尽量使用标准流。

◆标准流解决不了的使用浮动。

◆浮动解决不了的使用定位

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .father{
            width: 500px;
            height: 500px;
            background: red;
        }
        .son{
            width: 100px;
            height: 100px;
            background: green;
            /*设置盒子的左外边距为auto 将盒子向右冲*/
            margin-left: auto;
            /*设置盒子的右外边距为auto 将盒子向左冲*/
            margin-right: auto;
        }
    </style>
</head>
<body>

<div class="father">
    <div class="son"></div>
</div>
</body>
</html>
```

## （10）图片和文字的对齐

## １  图片和文字垂直居中对齐

vertical-align

对inline-block最敏感。默认属性是:vertical-align:baseline;

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		img{
			
			/*图片和文字垂直居中对齐*/
			vertical-align:middle;
		}
	</style>
</head>
<body>
	<div class="box">
	<img src="1.png" alt="">天太热了！
	</div>

</body>
</html>
```

![img](file:///C:/Users/huanghe/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

## （11）css的可见性

overflow:hidden;   溢出隐藏    

visibility:hidden;   隐藏元素    隐藏之后还占据原来的位置。

display:none;      隐藏元素    隐藏之后不占据原来的位置。

Display:block;     元素可见

Display:none    和display:block  常配合js使用

## （12）CSS内容的移出

使用text-indent:-5000em;

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.logo{
			width: 143px;
			height: 76px;
			background: url("logo.png");
		}
		a{
			display: inline-block;
			text-indent:-5000em;
		}
	</style>
</head>
<body>
	<div class="logo">
		<h1>	
		 <a href="#">搜狐</a>
		</h1>

	</div>

</body>
</html>
```

◆将元素高度设置为0,使用内边距将盒子撑开，给盒子使用overflow:hidden;将文字隐藏。

## （13）精灵图

