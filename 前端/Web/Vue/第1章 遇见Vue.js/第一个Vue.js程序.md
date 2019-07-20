# 第一个Vue.js程序

[TOC]

vue简介，vue是一个构建用户界面的框架。是一个轻量级mvv框架，和angular一样是所谓的双向数据绑定，数据驱动和组件化的前端开发，通过简单的api。实现响应式的数据绑定和组合试图组件，容易上手，小巧。

### 安装

我们可以在 Vue.js 的[官网](http://cn.vuejs.org/)上直接下载 vue.min.js 并用 `<script>` 标签引入。Vue 会被注册为一个全局变量。

> 重要提示：在开发时请用开发版本，遇到常见错误它会给出友好的警告。

代码示例

```html

<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>River-第一个Hello Vue程序</title>
	</head>
	<body>
		<div id="app">
			{{message}}
		</div>
		<script src="vue.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			var app=new Vue({
				el:'#app',
				data:{
					message:'Hello Vue!'
				}
			});
		</script>
	</body>
</html>
```

> 预览：<https://huanghe1993.github.io/chapter01/01Vue-hello.html>

页面输出内容：

<img src='http://man.hhaxmm.cn/blog/20190513/1kdIxCQbwPXT.png' align='left' />

### 第一个Hello Vue代码详解

> 1.将vue.js文件引入到当前页面

```js
<script src="vue.js" type="text/javascript" charset="utf-8"></script>
```

只要将vue.js文件引入页面，在当前环境就会多出一个全局变量：Vue

> 2.通过全局构造函数：Vue ，实例化一个Vue应用程序接管的元素（包括所有的子元素）

```js
<script type="text/javascript">
var app=new Vue({
  el:'#app', //el:element 的简写 ，用来指定Vue应用程序接管的元素（包括所有的子元素）
  data:{ //data:data就是Vue实例应用程序中的数据成员
  	message:'Hello Vue!'
  }
});
</script>
```

3.代码执行流程解析

> 1.浏览器从上到下依次进行解析
>
> ​		浏览器对于id=app 的div 内部的 {{message}}不认识，直接作为普通文本渲染到网页上
>
> 2.浏览器继续往后解析执行
>
> ​	   发现有一个js外链脚本，发起请求进行下载，当前页面环境拿到js脚本之后，vue.js就会执行，执行结束	   就向全局暴露出了一个对象：Vue
>
> 3.当解析执行到咱们自己的Script的时候，开始解析执行咱们自己的代码
>
> ​	3.1 创建Vue实例
>
> ​        通过 el 属性 指定 Vue程序 的接管范围
> ​       通过 data 向Vue 实例的应用程序中初始化了一个 message 成员
>
> ​    3.2 接下来
>
> ​     Vue 程序通过 el 属性指定id为 #app 的div
> ​     开始解析执行 Vue 能识别的语法
> ​     {{message}} 在Vue 中被称为双花括号插值表达式
> ​     在双括号插值表达式中可以使用当前元素所属Vue程序 接管范围的data中的数据

