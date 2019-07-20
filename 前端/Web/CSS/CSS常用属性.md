# CSS常用属性

## 伪类链接

```
a:link{属性:值;}  a{属性:值}效果是一样的

a:link{属性:值;}       链接默认状态	 
a:visited{属性:值;}     链接访问之后的状态 
a:hover{属性:值;}      鼠标放到链接上显示的状态  	a:active{属性:值;}      链接激活的状态
a:focus{属性:值；}     获取焦点
```

## 背景属性

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



## 边框属性

```
/*去除轮廓线*/
border:0 none;
/*去除边框*/
outline-style: none;
```



## 鼠标的属性

cursor:pointer; 小手的样式

​		text：输入文本的样式

​		Crosshair：十字交叉

​		move:移动的样式





## 定位属性

#### 静态定位

定位方向: left  | right  | top | bottom

◆position:static; 静态定位。默认值，就是文档流



#### 绝对定位

◆绝对定位

Position:absolute;

特点：

★元素使用绝对定位之后不占据原来的位置（脱标）

★元素使用绝对定位，位置是从浏览器出发。

★嵌套的盒子，父盒子没有使用定位，子盒子绝对定位，子盒子位置是从浏览器出发。

★嵌套的盒子，父盒子使用定位，子盒子绝对定位，子盒子位置是从父元素位置出发。

★给行内元素使用绝对定位之后，转换为行内块。（不推荐使用，推荐使用display:inline-block;）



#### 相对定位

◆相对定位

Position: relative;

特点： 

★使用相对定位，位置从自身出发。

★还占据原来的位置。

★**子绝父相（父元素相对定位，子元素绝对定位）******最后的子元素的位置是相对于父的

★行内元素使用相对定位不能转行内块



#### 固定定位

◆固定定位

Position:fixed;

特点：

★固定定位之后，不占据原来的位置（脱标）

★元素使用固定定位之后，位置从浏览器出发。

★元素使用固定定位之后，会转化为行内块（不推荐，推荐使用display:inline-block;） 



## 可见属性

overflow:hidden;   溢出隐藏    

visibility:hidden;   隐藏元素    隐藏之后还占据原来的位置。

display:none;      隐藏元素    隐藏之后不占据原来的位置。

Display:block;     元素可见

Display:none    和display:block  常配合js使用

```javascript
  //隐藏盒子
    box.onclick = function () {
        this.style.display = "none";
//        this.style.visibility = "hidden";
        this.style.opacity = "0";
//        this.style.position = "absolute";
//        this.style.top = "-50px";
    }
```



## InnerHtml  Value  innerText

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>

<input type="text" id="inp1" value="input标签"/>

<div id="box1">
    我是box1的内容
    <div id="box2">我是box2的内容</div>
</div>

<div id="box3">
    我是box1的内容
    <div id="box4">我是box2的内容</div>
</div>

<script>
    //value:标签的value属性
    console.log(document.getElementById("inp1").value);

    //innerHTML
    //innerHTML：获取双闭合标签里面的内容。（识别标签）
    console.log(document.getElementById("box1").innerHTML);
    /*
    获取的内容是：
     我是box1的内容
     <div id="box2">我是box2的内容</div>
     */
    document.getElementById("box1").innerHTML = "<h1>我是innerHTML</h1>";

    //innerText：获取双闭合标签里面的内容。（不识别标签）（老版本的火狐用textContent）
    console.log(document.getElementById("box3").innerText);
    document.getElementById("box3").innerText = "<h1>我是innerText</h1>";
</script>
</body>
</html>
```

