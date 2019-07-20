# Bootstrap

[TOC]



##Bootstrap 特点
Bootstrap 非常流行，得益于它非常实用的功能和特点。主要核心功能特点如下：
(1).跨设备、跨浏览器
可以兼容所有现代浏览器，包括比较诟病的 IE7、8。当然，本课程不再考虑 IE9 以下
浏览器。
(2).响应式布局
不但可以支持 PC 端的各种分辨率的显示，还支持移动端 PAD、手机等屏幕的响应式切
换显示。
(3).提供的全面的组件
Bootstrap 提供了实用性很强的组件，包括：导航、标签、工具条、按钮等一系列组
件，方便开发者调用。
(4).内置 jQuery 插件
Bootstrap 提供了很多实用性的 jquery 插件，这些插件方便开发者实现 Web 中各种
常规特效。
(5).支持 HTML5、CSS3
HTML5 语义化标签和 CSS3 属性，都得到很好的支持。
(6).支持 LESS 动态样式
LESS 使用变量、嵌套、操作混合编码，编写更快、更灵活的 CSS。它和 Bootstrap 能
很好的配合开发。 

##Bootstrap 结构
首先，想要了解 Boostrap 的文档结构，需要在官网先把它下载到本地。Bootstrap
下载地址如下：
Bootstrap 下载地址：http://v3.bootcss.com/ (选择生产环境即可，v3.3.4)
解压后，目录呈现这样的结构：
bootstrap/
├── css/
│ ├── bootstrap.css
│ ├── bootstrap.css.map
│ ├── bootstrap.min.css
│ ├── bootstrap-theme.css
│ ├── bootstrap-theme.css.map
│ └── bootstrap-theme.min.css
├── js/
│ ├── bootstrap.js
│ └── bootstrap.min.js
└── fonts/
├── glyphicons-halflings-regular.eot
├── glyphicons-halflings-regular.svg
├── glyphicons-halflings-regular.ttf
├── glyphicons-halflings-regular.woff
└── glyphicons-halflings-regular.woff2
主要分为三大核心目录：css(样式)、js(脚本)、fonts(字体)。
1.css 目录中有四个 css 后缀的文件，其中包含 min 字样的，是压缩版本，一般使用
这个；不包含的属于没有压缩的，可以学习了解 css 代码的文件；而 map 后缀的文件则是
css 源码映射表，在一些特定的浏览器工具中使用。
2.js 目录包含两个文件，是未压缩和压缩的 js 文件。
3.fonts 目录包含了不同后缀的字体文件。

## 创建第一个页面 

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Bootstrap介绍</title>
	<link rel="stylesheet" type="text/css" href="css/bootstrap.min.css">
	<script type="text/javascript" src="js/jquery-1.11.2.min.js"></script>
	<script type="text/javascript" src="js/bootstrap.min.js"></script>
</head>
<body>
	<button class="btn btn-info">Bootstarp</button>	
</body>
</html>
```

