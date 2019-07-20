# DOM和BOM_01

[TOC]

## (1)事件

JS是以事件驱动为核心的一门语言。

- [1.1  事件三要素]


事件源、事件、事件驱动程序。

三句话：获取事件源、绑定事件、书写事件驱动程序。

获取事件源：document.getElementById(“box”);

绑定事件：  box.onclick = function(){ 程序 };

书写事件驱动程序：以后要学习的关于DOM的操作

```javascript

	//获取事件源：document.getElementById(“box”);
	var div=document.getElementById("box")
	//绑定事件：  box.onclick = function(){ 程序 };
	div.onclick=function(){
		//书写事件驱动程序：以后要学习的关于DOM的操作
		alert(1);
	}
```

JS为我们提供的事件

| 事件名         | 说明                 |
| ----------- | ------------------ |
| onclick     | 鼠标单击               |
| ondblclick  | 鼠标双击               |
| onkeyup     | 按下并释放键盘上的一个键时触发    |
| onchange    | 文本内容或下拉菜单中的选项发生改变  |
| onfocus     | 获得焦点，表示文本框等获得鼠标光标。 |
| onblur      | 失去焦点，表示文本框等失去鼠标光标。 |
| onmouseover | 鼠标悬停，即鼠标停留在图片等的上方  |
| onmouseout  | 鼠标移出，即离开图片等所在的区域   |
| onload      | 网页文档加载事件           |
| onunload    | 关闭网页时              |
| onsubmit    | 表单提交事件             |
| onreset     | 重置表单时              |

### (1.2)获取事件源的方式

- 1.获取事件源：document.getElementById(“box”);
- 2.通过标签获取到var arr1=document.getElementsByTagName("div")[0]; 他的返回值是一个标签数组，习惯上是遍历之后再进行使用的;
- 3.通过类名获取到var arr2=document.getElementsByClassName("leiming")[0];     通过类名查找 HTML 元素在 IE5,6,7,8 中无效            他的返回值是一个标签数组，习惯上是遍历之后再进行使用的;
- 4.根据命名空间来获取元素var arr4=document.getElelmentsByTagNameNS("aaa")
- 5.根据名称来获取var arr3=document.getElementsByName("aaa");已经用的不多了

### （1.3）事件的绑定方式

- 第一种：匿名绑定

```javascript
div.onclick=function(){
		//书写事件驱动程序：以后要学习的关于DOM的操作
		alert(1);
	}
```

- 第二种：用函数名进行绑定

```javascript
//注意不能在fn后面加上（）
div.onclick=fn;
	function fn(){
		//书写事件驱动程序：以后要学习的关于DOM的操作
		alert(1);
	}
```

- 第三种：行内绑定

```html
//在这里面是当做方法来看的，让JS去获取这个方法
<div id="box" onclick="fn()"></div>
```

### （1.4）事件的驱动程序

可以操作标签的属性和样式

```javascript
div.className="aaa";  //修改类名
div.style.width="200px";
div.style.height="200px";
div.style.backgroundColor="red";
```
## (2)JS的加载问题

js的加载是和html同步进行加载的，（如果使用元素在定义元素之前容易报错）

**在整个页面中的所有的标签会当做对象来看待**，

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>

<img src="image2/1.jpg" alt="">
<div>huanghe</div>


<script >

	//网页文档加载事件
	//什么时候触动这个事件呢？页面加载完毕（文本和图片）的时候;会在页面或图像加载完成后立即发生
	//整个页面加载完毕再执行JS的内容（这里的加载只是DOM结构的加载）
	//防止使用元素在定义元素之前
	window.onload=function(){
		alert(1);
	}
</script>	
</body>
</html>
```

## （3）DOM概述

- 解析的过程

  HTML加载完毕，渲染引擎会在内存中把HTML文档，生成一个DOM树，getElementById是获取内中DOM上的元素节点。然后操作的时候修改的是该元素的属性。

- DOM是文档对象模型

  处理网页内容的方法和接口


- HTML组成的部分为结点、

  每一个HMTL标签都是一个元素节点（标签）

  标签中的文字则是文字节点。（文本）

  标签的属性是属性节点。（属性）

  ```html
  <!DOCTYPE html>
  <html>
  <head lang="en">
      <meta charset="UTF-8">
      <title></title>
  </head>
  <body>
  <div id="box" value="111">你好</div>

  <script>

      var ele = document.getElementById("box");//元素节点
      var att = ele.getAttributeNode("id");//属性节点
      var txt = ele.firstChild;//文本结点

      //    console.log(ele);
      //    console.log(att);
      //    console.log(txt);
      //nodeType
      console.log(ele.nodeType);//1
      console.log(att.nodeType);//2
      console.log(txt.nodeType);//3

      //nodeName
      console.log(ele.nodeName);//DIV
      console.log(att.nodeName);//id
      console.log(txt.nodeName);//#text

      //nodeValue
      console.log(ele.nodeValue);//null
      console.log(att.nodeValue);//box
      console.log(txt.nodeValue);//你好

  </script>
  </body>
  </html>
  ```

  ### （3.1）通过访问关系获取结点

  ![这里写图片描述](http://img.blog.csdn.net/20171117172209550?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

  ​

  ​	结点之间的访问关系，是以属性的方式存在的。

  ​	DOM的结点并不是孤立的，因此可以通过DOM结点之间的相对关系对它们进行访问

  #### （3.1.1）父节点（ParentNode）

  调用者就是节点。一个节点只有一个父节点.调用的方式就是parentNode.

  案例：

  1.通过子盒子设置父盒子的背景色

  2.关闭二维码

  ​

### （3.2）兄弟结点

Sibling就是兄弟的意思。

Next：下一个的意思。

Previous:前一个的意思。

nextSibling：调用者是节点。IE678中指下一个元素节点（标签）。在火狐谷歌IE9+以后都指的是下一个节点（包括空文档和换行节点）。

nextElementSibling：在火狐谷歌IE9都指的是下**一个元素节点**。

总结：在IE678中用nextSibling，在火狐谷歌IE9+以后用nextElementSibling

下一个兄弟节点=节点.nextElementSibling || 节点.nextSibling

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>

<div class="box1">
    <div class="box2"></div>
    <div class="box3"></div>
</div>
</body>
</html>
<script>
    //结点之间的关系是以属性的方式存在的

    //1.box1是box2的父盒子
    var box1=document.getElementsByClassName("box1")[0];
    var box2=document.getElementsByClassName("box2")[0];
    console.log(box2.parentNode);

    //netElementSibling:在火狐谷歌中指的是下一个兄弟元素结点
    console.log(box2.nextElementSibling);

    //firstElementChild获取的是第一个子孩子，都是获取的是元素结点不包括换行和空的文本结点
    var box2=box1.firstElementChild;
    console.log(box2);

    //lastElmentChild获取的是最后一个孩子
    var box3=box1.lastElementChild;
    console.log(box3);

    //childNodes，它返回的是指定元素的子元素的结合，包括HTML结点，所有的属性，文本结点
    var childs=box1.childNodes;
    console.log(childs);

    //children获取的就只是属性结点
    var childs2=box1.children;
    console.log(childs2);

    //随意的得到兄弟结点
    var x=box2.parentNode.children[1];
    console.log(x); //获取到的就是box3
</script>
```

### (3.3)补充知识：

节点自己.parentNode.children[index];随意得到兄弟节点。

作为了解内容：

function siblings(elm) {

​             var a = [];

​             var p = elm.parentNode.children;

​             for(var i =0,pl= p.length;i<pl;i++) {

​                    if(p[i]!== elm) a.push(p[i]);

​             }

​             return a;

}

定义一个函数。必须传递自己。定义一个数组，获得自己的父亲，在获得自己父亲的所有儿子（包括自己）。遍历所哟与的儿子，只要不是自己就放进数组中



## （4）DOM结点的操作

节点的访问关系都是属性。节点的操作都是函数或者方法



### （4.1）创建结点

•方式一：

–document.write()

•方式二：

–innerHTML

•方式三：

–createElement()

使用的方法是 document.createElement("标签名");

```
新的标签（节点） = document.createElement(“标签名”);
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>

<button>创建</button>

<ul>
    aaaa
</ul>

<script>
    //第一种创建方式：document.write();
    document.write("<li>我是document.write创建的</li>");

    var btn = document.getElementsByTagName("button")[0];
    var ul = document.getElementsByTagName("ul")[0];
//        btn.onclick = function () {
//            document.write("<li>我是document.write创建的</li>");
//        }
    //第二种：直接利用元素的innerHTNL方法。（innerText方法不识别标签）
    ul.innerHTML += "<li id='li1'>我是innerHTML创建的</li>"


    //第三种：利用dom的api创建元素
    var newLi = document.createElement("li");
    newLi.innerHTML = "我是createElement创建的";
    //    console.log(newLi);
    //在父元素的最后面添加元素。
    //    ul.appendChild(newLi);
    //指定位置添加
    var li1 = document.getElementById("li1");
    ul.insertBefore(newLi,li1);
</script>
</body>
</html>
```



### （4.2）插入结点

使用方法： 父节点.appendChild()

```
父节点.appendChild(新节点); 父节点的最后插入一个新节点
```



使用方法：父节点.insertBefore(要插入的节点，参考节点)

```
父节点.insertBefore(新节点,参考节点)在参考节点前插入;
```

如果参考节点为null，那么他将在节点最后插入一个节点



### （4.3）删除结点

用法：用父节点删除子节点

```
父节点.removeChild（子节点）；必须制定要删除的子节点
```

节点自己删除自己：

不知道父级的情况下，可以这么写：

node.parentNode.removeChild(node)



### （4.4）赋值结点

想要复制的节点调用这个函数cloneNode()，得到一个新节点。 方法内部可以传参，入股是true，深层复制，如果是false，只复制节点本身

```
新节点=要复制的节点.cloneNode(参数) ; 参数可选复制节点
```

用于复制节点， 接受一个布尔值参数， true 表示深复制（复制节点及其所有子节点）， false 表示浅复制（复制节点本身，不复制子节点）



### （4.5）结点属性（结点.属性）

获取：getAttribute(名称)

设置：setAttribute(名称, 值)

删除：removeAttribute(名称)

注意：IE6、7不支持。

调用者：节点。   有参数。   没有返回值。

每一个方法意义不同。