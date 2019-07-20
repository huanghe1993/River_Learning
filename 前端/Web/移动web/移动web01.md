# 移动web-01

[TOC]

# 第1章基础知识

## 1.1   屏幕

移动设备与PC设备最大的差异在于屏幕，这主要体现在屏幕尺寸和屏幕分辨率两个方面。

通常我们所指的屏幕尺寸，实际上指的是屏幕对角线的长度（一般用英寸来度量）如下图所示

​        ![mark](http://man.hhaxmm.cn/blog/20190511/Va5sD4Lb4cJn.png)                                          

而分辨率则一般用像素来度量 px，表示屏幕水平和垂直方向的像素数，例如1920*1080指的是屏幕垂直方向和水平方向分别有1920和1080个像素点而构成，如下图所示

   ![mark](http://man.hhaxmm.cn/blog/20190511/dBDNYkVv6MqT.png)

## 1.2   长度单位

在Web开发中可以使用px（像素）、em、pt（点）、in（英寸）、cm（厘米）做为长度单位，我们最常用px（像素）做为长度单位。

我们可以将上述的几种长度单位划分成**相对长度单位**和**绝对长度单位**。

![mark](http://man.hhaxmm.cn/blog/20190511/szxoIAK2XeB2.png)   

如上图所示，iPhone3G/S和iPhone4/S的屏幕尺寸都为3.5英寸（in）但是屏幕分辨率却分别为`480*320px`、`960*480px`，由此我们可以得出英寸是一个绝对长度单位，而像素是一个相对长度单位（像素并没有固定的长度）。

## 1.3   像素密度

DPI（Dots Per Inch）是印刷行业中用来表示打印机每英寸可以喷的墨汁点数，计算机显示设备从打印机中借鉴了DPI的概念，由于计算机显示设备中的最小单位不是墨汁点而是像素，所以用PPI（Pixels Per Inch）值来表示屏幕每英寸的像素数量，我们将PPI、DPI都称为像素密度，但PPI应用更广泛，DPI在Android设备比较常见。

如下图所示，利用 *勾股定理* 我们可以计算得出PPI

![mark](http://man.hhaxmm.cn/blog/20190511/Fkucl3BwrwA1.png)   

PPI值的越大说明单位尺寸里所能容纳的像素数量就越多，所能展现画面的品质也就越精细，反之就越粗糙。

==Retina即视网膜屏幕，苹果注册的命名方式，意指具有较高PPI（大于320）的屏幕。==

思考：在屏幕尺寸（英寸）固定时，PPI和像素大小的关系？

结论：屏幕尺寸固定时，当PPI 越大，像素的实际大小就会越小，当PPI越小，像素实际大小就越大。

## 1.4   设备独立像素

随着技术发展，设备不断更新，出现了不同PPI的屏幕共存的状态（如iPhone3G/S为163PPI，iPhone4/S为326PPI），像素不再是统一的度量单位，这会造成同样尺寸的图像在不同PPI设备上的显示大小不一样。

如下图，假设你设计了一个`163*163`的蓝色方块，在PPI为163的屏幕上，那这个方块看起来正好就是`1*1`寸大小，在PPI为326的屏幕上，这个方块看起来就只有`0.5*0.5`寸大小了。

   ![mark](http://man.hhaxmm.cn/blog/20190511/88smGbicwAwM.png)

做为用户是不会关心这些细节的，他们只是希望在不同PPI的设备上看到的图像内容差不多大小，所以这时我们需要一个新的单位，**这个新的单位能够保证图像内容在不同的PPI设备看上去大小应该差不多，这就是独立像素，在IOS设备上叫PT(Point)，Android设备上叫DIP(Device independent Pixel)或DP**。

举例说明就是iPhone 3G（PPI为163）1dp = 1px，iPhone 4（PPI为326）1dp = 2px。

   ![mark](http://man.hhaxmm.cn/blog/20190511/9tPjPtFxbhai.png)

我们也不难发现，如果想要iPhone 3G/S和iPhone 4/S图像内容显示一致，可以把iPhone 4/S的尺寸放大一倍（它们是一个2倍(@2x)的关系），即在iPhone3G/S的上尺寸为`44*44`px，在iPhone4/S上为`88*88px`，我们要想实现这样的结果可以设置`44*44d`p，这时在iPhone3G/S上代表`44*44px`，在iPhone4/S上代表`88*88px`，最终用可以看到的图像差不多大小。

> 通过上面例子我们不难发现dp同px是有一个对应（比例）关系的，这个对应（比例）关系是操作系统确定并处理，目的是确保不同PPI屏幕所能显示的图像大小是一致的，通过window.devicePixelRatio可以获得该比例值。

见代码示例1-1.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>获取独立像素与物理像素比例值</title>
</head>
<body>
<script>
    // 像素和设备独立像素的一个关系
    alert(window.devicePixelRatio);
</script>
</body>
</html>
```



下图展示了iPhone不同型号间dp和px的比例关系

![mark](http://man.hhaxmm.cn/blog/20190511/fBOsrw3JmI2v.png)   

从上图我们得知dp（或pt）和px并不总是绝对的倍数关系（并不总能保证能够整除），而是window.devicePixelRatio ~= 物理像素/独立像素，然而这其中的细节我们不必关心，因为操作系统会自动帮我们处理好（保证1dp在不同的设备上看上去大小差不多）。

> 虽然独立像素帮助我们解决了图片大小的问题，但是当在PPI较高的设备显示的时候，图片由于会进行放大，因此图片的显示质量会下降

## 1.5   像素

1、**物理像素**指的是屏幕渲染图像的最小单位，属于屏幕的物理属性，不可人为进行改变，其值大小决定了屏幕渲染图像的品质，**我们以上所讨论的都指的是物理像素**。

// 获取屏幕的物理像素尺寸

window.screen.width;

window.screen.height;

// 部分移动设备下获取会有错误，与移动开发无关，只需要了解

见代码示例1-2.html

2、**CSS像素**，与设备无关像素，指的是通过CSS进行网页布局时用到的单位，其默认值(PC端)是和物理像素保持一致的（1个单位的CSS像素等于1个单位的物理像素），但是我们可通缩放来改变其大小。

我们通过调整浏览器的缩放比例可以直观的理解CSS像素与物理像素之前的对应关系，如下图所示：

   ![mark](http://man.hhaxmm.cn/blog/20190511/bA7TBcmR3sWE.png)

> 我们需要理解的是物理像素和CSS像素的一个关系，1个物理像素并不总是等于一个CSS像素，通过调整浏览器缩放比例，可以有以上3种情况。

# 第2章远程调试

## 2.1   模拟调试

现代主流浏览器均支持移动开发模拟调试，通常按F12可以调起，其使用也比较简单，可以帮我们方便快捷定位问题。

## 2.2   真机调试

模拟调试可以满足大部分的开发调试任务，但是由于移动设备种类繁多，环境也十分复杂，模拟调试容易出现差错，所以真机调试变的非常必要。

有两种方法可以实现真机调试：

1、将做好的网页上传至服务器或者本地搭建服务器，然后移动设备通过网络来访问。（重点）

2、借助第三方的调试工具，如weinre、debuggap、ghostlab(比较)等

真机调试必须保证移动设备同服务器间的网络是相通的。

# 第3章 视口

视口（viewport）是用来约束网站中最顶级块元素`<html>`的，即它决定了`<html>`的大小。

## 3.1   PC设备

在PC设备上viewport的大小取决于浏览器窗口的大小，以CSS像素做为度量单位。

通过以往CSS的知识，我们都能理解`<html>`的大小是会影响到我们的网页布局的，而viewport又决定了`<html>`的大小，所以viewport间接的决定并影响了我们网页的布局。

// 获取viewport的大小

document.documentElement.clientWidth;

document.documentElement.clientHeight;

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>视口</title>
    <style>
        body {
            padding: 0;
            margin: 0;
            background-color: #F7F7F7;
        }
    </style>
</head>
<body>

<script>
    // 获取到html元素的大小
    var clientWidth = document.documentElement.clientWidth;
    var clientHeight = document.documentElement.clientHeight;

    console.log('PC设备Viewport的宽度为：' + clientWidth);
    console.log('PC设备Viewport的高度为：' + clientHeight);
</script>
</body>
</html>
```



下面我们通过一个代码实例来演示PC设备viewport（浏览器窗口）是如何影响我们的网页布局的，见代码示例3-2.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>视口</title>
	<style>
		body {
			padding: 0;
			margin: 0;
			background-color: #F7F7F7;
		}

		.info {
			line-height: 20px;
			padding: 10px;
			background-color: pink;
		}

		.info span {
			display: block;
		}

		.viewport {
			/*1.如果确定具体的宽高 值，当超出viewport的大小的时候，会出现滚动条
			2.如果设置的宽度为100%,当子元素宽高大于父容器的时候，会自动换行
			3.如果不想出现滚动条或者换行，可以将子元素设置为父容器的百分比*/
			width: 100%;
			/*width: 1152px;*/
			height: 200px;
			background-color: #CCC;
		}

		.viewport .box {
			/*width: 288px;*/
			width: 25%;
			height: 200px;
			text-align: center;
			line-height: 200px;
			font-size: 28px;
			float: left;
			background-color: blue;
			border-right: 2px solid #ccc;
			box-sizing: border-box;
		}
	</style>
</head>
<body>
	<!-- 显示窗口信息 -->
	<div class="info">
		<span class="width"></span>
		<span class="height"></span>
	</div>

	<div class="viewport">
		<div class="box">1</div>
		<div class="box">2</div>
		<div class="box">3</div>
		<div class="box">4</div>
	</div>
	<!-- 一个类似jQuery的库 -->
	<script src="../js/zepto.js"></script>
	<script>

		var clientWidth, clientHeight;
		var width = $('.width'),
			height = $('.height');

		// 用来获取viewport
		function getSize() {
			clientWidth = document.documentElement.clientWidth;
			clientHeight = document.documentElement.clientHeight;

			width.text('PC设备Viewport的宽度为：' + clientWidth);
			height.text('PC设备Viewport的高度为：' + clientHeight);
		}

		// 调用
		getSize();

		// 监听窗口变化
		window.onresize = function () {
			getSize();
		}
	</script>
</body>
</html>
```



当我们调整浏览器窗口时，4个浮动的盒子换行显示了，原因是父盒子宽度不足以容纳4个子盒子，要解决这个问题可以给父盒子设置一个固定（较大）的宽度，如1152px，见代码示例3-2.html这样我们可以解决盒子不换行的问题，也可以保证我们的网页内容可以正常的显示，但是出现了滚动条。

> 注：所谓正常显示是指页面布局没有错乱。

总结：在PC端，我们通过调整浏览器窗口可以改变viewport的大小，为了保证网页布局不发生错乱，需要给元素设定较大固定宽度。

分析完PC端的情况，我们再来看一下移动端会是什么情况呢？

## 3.2   移动设备

移动设备屏幕普遍都是比较小的，但是大部分的网站又都是为PC设备来设计的，要想让移动设备也可以正常显示网页，移动设备不得不做一些处理，通过上面的例子我们可以知道只要viewport足够大，就能保证原本为PC设备设计的网页也能在移动设备上正常显示，移动设备厂商也的确是这样来处理的。

在移动设备上viewport不再受限于浏览器的窗口，而是允许开发人员自由设置viewport的大小，通常浏览器会设置一个默认大小的viewport，为了能够正常显示那些专为PC设计的网页，一般这个值的大小会大于屏幕的尺寸。

如下图为常见默认viewport大小（仅供参考）

![mark](http://man.hhaxmm.cn/blog/20190511/r3HIr9q5o6qk.png)                                                  

从图中统计我们得知不同的移动厂商分别设置了一个默认的viewport的值，这个值保证大部分网页可以正常在移动设备下浏览。

下面我们通过一个小示例来验证上述结论，执行环境为iPhone5s

```html
<!--
 * @Description: 
 * @Author: river
 * @LastEditors: huanghe
 * @Date: 2019-05-11 19:06:27
 * @LastEditTime: 2019-05-11 21:17:49
 -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>视口</title>
    <style>
        body {
            padding: 0;
            margin: 0;
            background-color: #F7F7F7;
        }

        .box {
            width: 490px;
            height: 200px;
            text-align: center;
            line-height: 200px;
            background-color: pink;
            float: left;
        }
    </style>
</head>
<body>
<div class="box">1</div>
<div class="box">2</div>
</body>
</html>
```

在示例中我们设定.box{width: 490px;}了，发现两个盒子正好一行显示，见代码示例

![mark](http://man.hhaxmm.cn/blog/20190511/gn1xdQOpcnsq.png)

设定了.box{width: 491px;}，则换行显示了，见代码示例3-5.html

比较可以验证iPhone5s下viewport的默认宽为980px，这样便可以保证原本为PC设计的网页，在移动端上也不会发生布局的错乱。

注：早期网页一般宽度设计成960px。

我们再做另一个测试，见代码示例3-6.html

> 在iPhone5s和部分Android中我们发现页面内容（文字、图片）被缩放了（变的非常小），而在部分安卓设备中则出现了滚动条。

产生缩放和滚动条的原因是什么呢？

> 要解释上面的原因，需要进一步对移动设备的viewport进行分析，移动设备上有2个viewport（为了方便讲解人为定义的），分别是layout viewport和ideal viewport。

1、layout viewport（布局视口）指的是我们可以进行网页布局区域的大小，同样是以CSS像素做为计量单位，可以通过下面方式获取

// 获取layout viewport

document.documentElement.clientWidth;

document.documentElement.clientHeight;

通过前面介绍我们知道，如果要保证为PC设计的网页在移动设备上布局不发生错乱，移动设备会默认设置一个较大的viewport（如IOS为980px），这个viewport实际指的是layout viewport。

见示例代码3-7.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>视口</title>
	<style>
		body {
			padding: 0;
			margin: 0;
			background-color: #F7F7F7;
		}

		.box {
			height: 300px;
			text-align: center;
			background-color: pink;
		}
	</style>
</head>
<body>
<script src="../js/zepto.js"></script>
<!-- 显示窗口信息 -->
<div class="info">
	<span class="width"></span>
	<span class="height"></span>
</div>
	<div class="box">layout viewport</div>
	<script>
		var clientWidth, clientHeight;
		var width = $('.width'),
				height = $('.height');
		// 用来获取viewport
		function getSize() {
			/*默认的视口大小*/
			clientWidth = document.documentElement.clientWidth;
			clientHeight = document.documentElement.clientHeight;

			width.text('PC设备Viewport的宽度为：' + clientWidth);
			height.text('PC设备Viewport的高度为：' + clientHeight);
		}

		// 调用
		getSize();


		var clientWidth = document.documentElement.clientWidth;
		var clientHeight = document.documentElement.clientHeight;

		console.log('layout viewport 的宽度为：' + clientWidth);
		console.log('layout viewport 的高度为：' + clientHeight);
	</script>
</body>
</html>
```



2、ideal viewport（理想视口）设备屏幕区域，（以设备独立像素PT、DP做为单位）以CSS像素做为计量单位，其大小是不可能被改变，通过下面方式可以获取。

// 获取ideal viewport有两种情形

// 新设备

window.screen.width;

window.screen.height;

// 老设备

window.screen.width / window.devicePixelRatio;

window.screen.height / window.devicePixelRatio;

见代码示例3-8.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>视口</title>
	<style>
		body {
			padding: 0;
			margin: 0;
			background-color: #F7F7F7;
		}

		.box {
			height: 300px;
			text-align: center;
			background-color: pink;
		}
	</style>
</head>
<body>
<script src="../js/zepto.js"></script>
<!-- 显示窗口信息 -->
<div class="info">
	<span class="width"></span>
	<span class="height"></span>
</div>
	<div class="box">ideal viewport</div>
	<script>
		var clientWidth, clientHeight;
		var width = $('.width'),
				height = $('.height');
		// 用来获取viewport
		function getSize() {
			clientWidth = window.screen.width;
			clientHeight = window.screen.height;

			width.text('PC设备Viewport的宽度为：' + clientWidth);
			height.text('PC设备Viewport的高度为：' + clientHeight);
		}

		// 调用
		getSize();
		var screenWidth = window.screen.width;
		var screenHeight = window.screen.height;

		console.log('ideal viewport 的宽度为：' + screenWidth);
		console.log('ideal viewport 的高度为：' + screenHeight);

		console.log('设备像素比例为：' + window.devicePixelRatio);

		// 1dp/pt = 2px

		// 640px / 2 = dp/pt

		// ideal 是我们的屏幕大小
		// 屏幕大小是以独立像素做为单位的
		
	</script>
</body>
</html>
```

并不总是正确的，然而在实际开发我们一般无需获取这个值具体大小。

理解两个viewport后我们来解释为什么网页会被缩放或出现水平滚动条，其原因在于移动设备浏览器会默认设置一个layout viewport，并且这个值会大于ideal viewport，那么我们也知道ideal viewport就是屏幕区域，layout viewport是我们布局网页的区域，那么最终layout viewport是要显示在ideal viewport里的，而layout viewport大于ideal viewport时，于是就出现滚动条了，那么为什么有的移动设备网页内容被缩放了呢？移动设备厂商认为将网页完整显示给用户才最合理，而不该出现滚动条，所以就将layout viewport进行了缩放，使其恰好完整显示在ideal viewport（屏幕）里，其缩放比例为ideal viewport / layout viewport。

## 1.3   移动浏览器

移动端开发主要是针对IOS和Android两个操作系统平台的，除此之外还有Windows Phone。

移动端主要可以分成三大类，系统自带浏览器、应用内置浏览器、第三方浏览器

系统浏览器：指跟随移动设备操作系统一起安装的浏览器。

应用内置浏览器：通常在移动设备上都会安装一些APP例如QQ、微信、微博、淘宝等，这些APP里往往会内置一个浏览器，我们称这个浏览器为应用内置浏览器（也叫WebView），这个内置的浏览器一般功能比较简单，并且客户端开发人员可以更改这个浏览器的某些设置，在我们理实的开发里这个浏览器很重要。

第三方浏览器：指安装在手机的浏览器如FireFox、Chrome、360等等。

在IOS和Android操作系统上自带浏览器、应用内置浏览器都是基于Webkit内核的。

思考：移动端页面要达到什么效果才最合理？

# 第4章屏幕适配

经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置`<meta name="viewport" content="">`来进行控制，并改变浏览器默认的layout viewport的宽度。

## 1.1   Viewport详解

viewport 是由苹果公司为了解决移动设备浏览器渲染页面而提出的解决方案，后来被其它移动设备厂商采纳，其使用参数如下：

// 通过设置属性content=""实现，中间以逗号分隔

// 例如`<meta name="viewport" content="width=device-width">`

>  width 设置layout viewport 宽度，其取值可为数值或者device-width。
>
>  height 设置layout viewport 高度，其取值可为数值或者device-height
>
>  initital-scale设置页面的初始缩放值，为一个数字，可以带小数。
>
>  maximum-scale允许用户的最大缩放值，为一个数字，可以带小数。
>
>  minimum-scale允许用户的最小缩放值，为一个数字，可以带小数。
>
>  user-scalable是否允许用户进行缩放，值为"no"或"yes"。

注：device-width 和 device-height就是ideal viewport的宽高。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!--对viewport的设置的meta建议写在已有的meta标签(<meta charset="UTF-8">)之后-->
    <!--name="viewport":说明当前meta标签是用来设置viewport的属性的，这个属性只有在移动端才会有效-->
    <!--content="":进行viewport的设置
    width:设置viewport的宽度  device-width:获取当前设备的宽度
    initial-scale=1:设置初始缩放比例  当我们设置width=device-width，也达到了initial-scale=1的效果，得知其实 initial-scale = ideal viewport / layout viewport  意味着，如果initial-scale设置为1，相当于设置了两个viewport的宽度一致
    maximum-scale:设置最大的缩放比例
    minimum-scale:设置默认状态下最小的缩放比例-->
    <!--<meta name="viewport" content="width=device-width">-->
    <!--<meta name="viewport" content="initial-scale=1
    user-scalable:设置是否允许用户进行缩放yes/no">-->

    <!--为了达到兼容：-->
    <!--<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,user-scalable=no">-->
    <!--meta:vp+tab-->
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <title>Title</title>
    <style>
        body{
            padding: 0;
            margin: 0;
        }
        div{
            width: 100%;
            height: 200px;
            background-color: pink;

        }
    </style>
</head>
<body>
<div>经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置来进行控制，并改变浏览器默认的layout viewport的宽度。经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置来进行控制，并改变浏览器默认的layout viewport的宽度。经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置来进行控制，并改变浏览器默认的layout viewport的宽度。经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置来进行控制，并改变浏览器默认的layout viewport的宽度。经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置来进行控制，并改变浏览器默认的layout viewport的宽度。经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置来进行控制，并改变浏览器默认的layout viewport的宽度。经过分析我们得到，移动页面最理想的状态是，避免滚动条且不被默认缩放处理，我们可以通过设置</div>
</body>
</html>
```





## 1.2   控制缩放

​    1、设置<meta name="viewport" content="initial-scale=1">，这时我们发现网页没有被浏览器设置缩放。见代码示例4-3.html

2、设置<meta name="viewport" content="width=device-width">，这时我们发现网页也没有被浏览器设设置缩放。见代码示例4-1.html

当我们设置width=device-width，也达到了initial-scale=1的效果，得知其实 initial-scale = ideal viewport / layout viewport。

两种方式都可以控制缩放，开发中一般同时设置width=device-width和initial-scale=1.0（为了解决一些兼容问题）参见[移动前端开发之viewport深入理解](http://www.cnblogs.com/2050/p/3877280.html)，即<meta name="viewport" content="width=device-width, initial-scale=1.0">

## 1.3   避免滚动

我们知道，滚动条是 layout viewport 相对于 ideal viewport 的，所以只要设置 layout viewport 小于或等于 ideal viewport，即<meta name="viewport" content="width=device-width">。

经测试发现我们并没有完全的解决滚动条的问题，原因在于我们示例里的.box {width: 490px;}设置了一个绝对的宽度造成的，要解决这个问题我们可以设置一个百分比（100%）的宽度，见代码示例4-7.html

## 1.4   适配方案

移动开发的核心是屏幕适配，然而并示有专门的规范进行约束，一般是对现有持术进行归纳而总结出适配方案，掌握了以上的技术细节后我们可以总结出以下几种适配方案：

**1.**    

**2.**    

**3.**    

**4.**    

**4.1.**    

**4.2.**    

**4.3.**    

**4.4.**    

### 4.4.1.   固定宽度

1、设置<meta name="viewport" content="width=device-width, initial-scale=1">

2、设置内容区域大小为320px

3、设置内容区域水平居中显示

关于手机尺寸（ideal viewport）

​                                                  

[更多ideal viewport参考](http://www.viewportsizes.com/)

通过汇总对比我们知道移动设备的屏幕尺寸虽然庞杂，但有几个主要尺寸，分别为320px、360px，这三个尺寸占了绝大部分，并且以320px最多，所以我们移动网页如果设计成320px宽，则可以保证在绝大多数设备里正常显示，此方案已经很少采用了。

见示例代码4-8.html

### 4.4.2.   百分比宽度

1、设置<meta name="viewport" content="width=device-width, initial-scale=1">

2、设置页面宽度为百分比

我们需要重新认识CSS里百分比的使用，见代码示例4-9.html

// 测试下列属性设置为百分比

width       参照父元素的宽度

height           参照父元素的高度

padding  参照父元素的宽度

border     不支持百分比设置

margin    参照父元素的宽度

我们发现这种方案最容易理解，但是在设置元素高度时有非常大的局限性。

### 4.4.3.   rem单位

1、设置<meta name="viewport" content="width=device-width, initial-scale=1">

\2. 设置页面元素宽度单位为rem 或 em

注：此方案比较灵活，我们的案例将采用这种方案

关于em和rem

em 相对长度单位，其参照当前元素字号大小，如果当前元素未设置字号则会继承其祖先元素字号大小例如 .box {font-size: 16px;}则 1em = 16px .box {font-size: 32px;} 则 1em = 32px，0.5em = 16px

见代码示例4-10.html

rem 相对长度单位，其参照根元素(html)字号大小例如 html {font-size: 16px;} 则 1rem = 16px html {font-size: 32px;} 则 1rem = 32px，0.5rem = 16px

见代码示例4-11.html

### 4.4.4.   100%像素

1、设置网页宽度等于设备物理像素

2、设置初始化缩放比例（值为1 / window.devicePixelRatio）

淘宝针对iPhone设备采用的这种方案