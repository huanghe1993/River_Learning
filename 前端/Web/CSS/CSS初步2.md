# CSS初步2（块，行内元素）（三大特性）

[TOC]

## (1)标签的分类

### （1.1）块元素

典型的代表Div,h1-h6,p,ul,li

特点: ★独占一行

​          ★可以设置宽高

​	  ★嵌套（包含）下，子块元素宽度（没有定义情况下）和父块元素宽度默认一致

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		div{
			height:200px;
			width:500px;
			background-color: red;
		}
		p{
			height:200px;
			background-color: green;
		}
		div.div1{
			height: 200px;
			width: 500px;
			background-color: #888;
		}
		.div1 .p1{
			height: 100px;
			background-color: red;
		}
	</style>
</head>
<body>
	<div>黄河的html5</div>
	<p>湖北省孝感市孝南区</p>
	<div class="div1">
		<p class="p1"></p>
	</div>
</body>
</html>
```

### （1.2）行内元素

```
<span></span><a href="#">黄河</a><strong></strong>
```

典型代表 span  ,a,  ,strong , em, del,  ins

特点：★在一行上显示

​      ★不能直接设置宽高

​      ★元素的宽和高就是内容撑开的宽高（一个汉字是16PX）

### （1.3）行内块元素（内联元素）

典型代表  input  img

特点：★在一行上显示

​           ★可以设置宽高

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		img{
			width: 300px;
		}
		input{
			width: 300px;
			height: 200px;
			background-color: red;

		}
	</style>
</head>
<body>
	<img src="images/a.jpg" alt=""><input type="text" name="">
</body>
</html>
```

## （2）块元素和行内元素的转换

#### （2.1）块元素和行内元素的相互的转换

display: inline;

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		div,p{
			display: inline;
		}
	</style>
</head>
<body>
	<div>湖北省孝感市</div>
	<p>陕西省西安市</p>
</body>
</html>
```

#### （2.2）行内元素和块元素的转换

```
display:block;
```

### （2.3）块和内元素相互转化为行内快元素

```
display:inline-block;
```

## （3）CSS的三大特性

### （3.1）层叠性

当多个样式作用于同一个（同一类）标签时，样式发生了冲突，总是执行后边的代码(后边代码层叠前边的代码)。和标签调用选择器的顺序没有关系

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.box2{
			font-size: 40px;
			color: red;
		}
		.box{
			font-size: 80px;
			color: red;
		}
	</style>
</head>
<body>
	<div class="box box2">huangeh</div>
</body>
</html>
```

这个代码执行的结果就是.box的css的样式代码执行的结果

### （3.2）继承性

继承性发生的前提是包含（嵌套关系）

★文字颜色可以继承

   ★文字大小可以继承

   ★字体可以继续

   ★字体粗细可以继承

   ★文字风格可以继承

   ★行高可以继承

总结：文字的所有属性都可以继承

◆特殊情况：

h系列不能继承文字大小。但是可以继承的是文字的颜色

a标签不能继承文字颜色。但是可以继承文字的大小

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.father{
			color:red;
			font: italic 700 40px microsoft yahei;
		}
		.box{
			font-size: 30px;
			color:yellow;
		}
	</style>
</head>
<body>
	<div class="father">
		<p>14期威武</p>
	</div>
	<div class="box">
		<h1>14期威武</h1>
		<h2>14期威武</h2>

		<p>14期霸气</p>
	</div>
	<div class="box">
		<a href="#">14期高薪</a>
	</div>
</body>
</html>
```

### （3.3）优先级

 默认样式<标签选择器<类选择器<id选择器<行内样式<!important  

​         0         1          10    100      1000      1000

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
	#con{
			color: pink;
			font-size:100px;
		}
		
	.box{
			color:green;
			font-size: 60px;
		}

	div{
			color:red !important;
			font-size:60px !important;
		}
		
	</style>
</head>
<body>
	<div class="box" id="con" style="font-size:12px; color:yellow;">14期威武</div>
</body>
</html>
```

优先级的特点：

​	(1)继承的权重为0;(当它自己没有定义样式的时候，它的样式是继承他父类的。但是它自己有了不管他的权重为什么，他都是用他自己的)

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.father{
			font-size: 60px;
			color: red;
		}
		p{
			font-size: 20px;
			color: blue;
		}
		#box{
			font-size: 90px;
			color:green;
		}
	</style>
</head>
<body>
	<div class="father" id="box">
		<p>14期威武</p>
	</div>
</body>
```

最终这个代码显示的是蓝色字体20px;

(2)继承的权重会出现叠加

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
	   p.son{
			font-size: 120px;
			color: yellow;
		}
		p{    
			font-size:30px;
			color:red;
		}
		.father .son{        
			font-size:500px;
			color: pink;
		}
		.son{
			font-size: 60px;
			color: blue;
		}
		.father #baby{
			font-size:12px;
			color: orange;
		}

	</style>
</head>
<body>
	<div class="father">
		<p class="son" id="baby">14期威武</p>
	</div>
</body>
</html>
```

## （4）伪类链接

```html
a:link{属性:值;}  a{属性:值}效果是一样的

a:link{属性:值;}       链接默认状态	 
a:visited{属性:值;}     链接访问之后的状态 
a:hover{属性:值;}      鼠标放到链接上显示的状态  	a:active{属性:值;}      链接激活的状态
  ：focus{属性:值；}     获取焦点
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		/*a:link{
			color: red;
		}*/
		/*链接默认状态*/
		a{
			color: red;
			text-decoration: none;
		}
		/*链接访问之后的状态*/
		a:visited{
			color: green;
		}
		/*鼠标放到链接上显示的状态*/
		a:hover{
			color: blue;
			text-decoration: line-through;
		}
		/*链接激活的状态*/
		a:active{
			color: pink;
		}


	</style>
</head>
<body>
	<a href="#">14期霸气</a>
</body>
</html>
```

**文本修饰属性**

text-decoration:none  |  underline   |     line-through

## （五）背景属性

| background-color                         | 背景颜色                            |
| ---------------------------------------- | ------------------------------- |
| background-image：url("1.jpg")            | 背景图片                            |
| background-repeat\|no-repeated\|repeated-x\|repeated-y | 背景平铺                            |
| background-position left\|right\|center\|top\|bottom | 背景定位                            |
| background-position:right                | 方位写一个的时候，另外的默认居中                |
| background-position:right bottom         | 写2个方位值的时候，顺序没有要求                |
| background-position:20px 30px            | 写2个具体值的时候，第一个值代表水平方向，第二个值代表垂直方向 |
| Background-attachment                    | 是否滚动 scroll  fixed(就是以浏览器进行定位的) |
| background:red url("1.jpg") no-repeated 30px 40px scroll; | 背景属性的连写                         |

未完待续。。。。。。。。。。。。