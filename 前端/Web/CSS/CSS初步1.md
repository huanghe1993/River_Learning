# CSS初步1（选择器）

[TOC]



CSS 指层叠样式表 (Cascading Style Sheets)(级联样式表)

书写的位置

- 内嵌式

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		
	</style>
</head>
<body>
	
</body>
</html>
```

- 外链式

```
<link rel=”stylesheet” href=”1.css”>
```

- 行内样式

```
<h1 style="font-size: 20px;color: green;">黄河的html5</h1>
```

## （1）选择器

选择器是一个选择谁（标签）的过程。

选择器（属性：值;）

| 属性                                 | 解释        |
| ---------------------------------- | --------- |
| Width:20px;                        | 宽         |
| Height:20px;                       | 高         |
| Background-color:red;              | 背景颜色      |
| font-size:24px;                    | 文字大小      |
| text-align:left \| center\|  right | 内容的水平对齐方式 |
| text-indent:2em;                   | 首行缩进      |
| Color:red;                         | 文字颜色      |

| 颜色的显示方式：                                 |                         |
| ---------------------------------------- | ----------------------- |
| （1）直接写颜色的名称                              |                         |
| （2）十六进制显示颜色#000000是黑色，前两位是红色，中间两位是绿色，最后的两位是蓝色 | 值越接近f表示颜色越浅，越接近0表示的就是越深 |
| （3）rgb(0,0,0，a)最后的一个代表的是透明度0是不透明         | 当三个的浓度一样的时候就是灰色         |

| 文字的属性                                    |                                     |
| ---------------------------------------- | ----------------------------------- |
| font-size:16px"                          | 文字的大小                               |
| font-weight:700\|bold(加粗)                | 文字的粗细，值从100-900，从700开始加粗(不推荐使用bold） |
| font-family:微软雅黑                         | 文字的字体                               |
| font-style:normal（italic）                | normal是正常,italic是倾斜的意思              |
| line-height：40px                         | 行高                                  |
| 文字属性的简写：                                 |                                     |
| Font:font-style font-weight font-size/line-height |                                     |
| Font:italic 700 16px/40px 微软雅黑           |                                     |



### (1.1)基础选择器

#### （1.1.1）标签选择器

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
	/*标签选择器*/
		div{
			font-size: 50px;
			color: green;
			background-color: yellow;
			width: 300px;
			height: 200px; 
		}
		p{
			color: pink;
			font-size: 60;
		}
	</style>
</head>
<body>
	<div>黄河</div>
	<div>hello world</div>
	<div>hello world</div>
	<p>java</p>
	<p>java</p>
	<p>java</p>
</body>
</html>
```



#### （1.1.2）类选择器

```
.自定义类名{属性：值；}

特点： 谁调用，谁生效。
        一个标签可以调用多个类选择器。当属性有冲突的时候使用的是后面的类名
       多个标签可以调用同一个类选择器。
类选择器命名规则
   ◎不能用纯数字或者数字开头来定义类名
   ◎不能使用特殊符号或者特殊符号开头（_）来定义类名
   ◎不建议使用汉字来定义类名
   ◎不推荐使用属性或者属性的值来定义类名

```

![这里写图片描述](http://img.blog.csdn.net/20171108111446222?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		#box{
			font-size: 30px;
			color: rgb(0,0,255);
			background-color: rgb(255,255,0);

		}
		.miss{
			font-size: 40px;
			color: #ff0000;
			background-color: #999;
		}
		.fox{
			text-indent: 2em;
			color: #00ff00;
		}
	</style>
</head>
<body>
	<div class="miss" id="box">黄河</div>
	<div class="miss fox">hello world</div>
	<div>hello world</div>
	<p>java</p>
	<p>java</p>
	<p>java</p>
</body>
</html>
```



#### （1.1.3）ID选择器

```
#自定义名称{属性：值；}
```

一个id选择器在一个页面只能使用一次，如果使用了2次或者是2次以上不符w3c的规范，JS调用会出问题

一个标签只能调用一个ID选择器。
一个标签可以同时调用类选择器和ID选择器，如果是有重复的属性的情况下，有限的使用ID的选择器

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		#box{
			font-size: 30px;
			color: rgb(0,0,255);
			background-color: rgb(255,255,0);

		}
	</style>
</head>
<body>
	<div id="box">黄河</div>
	<div>hello world</div>
	<div>hello world</div>
	<p>java</p>
	<p>java</p>
	<p>java</p>
</body>
</html>
```

#### （1.1.4）通配符选择器

```
 1. div是块级元素， 实际上就是一个区域， 主要用于容纳其他标签。 默认的display属性是block
 2. span是行内元素， 主要用于容纳文字。 默认的display属性是inline
 看看如下：
div占用的位置是一行，
span占用的是内容有多宽就占用多宽的空间距离，说明如下图
```
```
*{属性：值;}；
```

不推荐使用，增加浏览器和服务器负担

### (1.2)复合选择器

#### （1.2.1）交集选择器

两个或者是两个以上的选择器通过不同的方式连接在一起就构成了符合选择器

```
标签+类（ID）选择器{属性：值}
特点：即要满足使用了某个标签，还要满足使用了类（id）选择器
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.box{
			font-size: 50px;
		}
		/*标签+类选择器*/
		div.box{
			color: #ff0000;
		}
	</style>
</head>
<body>
	<div class="box">huangeh</div>
	<p class="box">hello word</p>
	<div>huanghe</div>
</body>
</html>
```



#### （1.2.2）后代选择器

```html
选择器+空格+选择器（属性：值）

后代选择器首选要满足包含（嵌套）关系，父集元素在前边，子集元素在后边

特点：无限制隔代。
      只要能代表标签，标签、类选择器、ID选择器自由组合

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		.box{
			font-size: 50px;
			color: red;
		}
		div span{
			font-size: 80px;
		}
		.box span{
			background-color: blue;
		}
		.box .miss{
			color: yellow;
		}
	</style>
</head>
<body>
	<div class="box">
		<p><span class="miss">湖北省孝感市</span></p>
	</div>
	<div class="box"><span>湖北省</span></div>
</body>
</html>
```

#### （1.2.3）子代选择器

```
选择器 > 选择器{属性：值;}

选择的是直接的下一代而不是间接的下一代
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		div>span{
			color: red;
			font-size: 40px;
		}
		p>span{
			font-size: 60px;
		}
	</style>
</head>
<body>
	<div>
		<!-- span是他的间接的选择器 -->
		<p><span>湖北省</span></p>
		<!-- span是他直接的子代选择器 -->
		<span>湖北省</span>
	</div>
</body>
</html>
```

#### （1.2.4）并集选择器

```
选择器+，+选择器+，选择器{属性:值;}

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		div,p,span,h1{
			font-size: 60px;
			color: #666;
			background-color: green;
		}
	</style>
</head>
<body>
	<div>湖北省</div>
	<p>孝感市</p>
	<span>孝南区</span>
	<h1>黄河</h1>
</body>
</html>
```