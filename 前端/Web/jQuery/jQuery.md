# jQuery

[TOC]

## 1. 介绍

jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（或JavaScript框架）。jQuery设计的宗旨是“write Less，Do More”，即倡导写更少的代码，做更多的事情。它封装JavaScript常用的功能代码，提供一种简便的JavaScript设计模式，优化HTML文档操作、事件处理、动画设计和Ajax交互。

jQuery它是javascript的一个轻量级框架，对javascript进行封装，它提供了很多方便的选择器。供你快速定位到需要操作的元素上面去。还提供了很多便捷的方法

jQuery最大的特点是：浏览器兼容，链式编程，隐式迭代，丰富的选择器操作。

### 1.1  jQuery环境

Jquery它是一个JavaScript类库，要想使用它，必须先引入；jQuery库的类型分为两种，分别是生产版（最小化和压缩版）和开发版（未压缩版），它们的区别如表1-1所示。

​                                  **表1-1　几种jQuery库类型对比**

|        名　　称         | 大　　小 | 说　　明                                                  |
| :---------------------: | -------- | --------------------------------------------------------- |
|   jquery.js（开发版）   | 约229 KB | 完整无压缩版本，主要用于测试、学习和开发                  |
| jquery.min.js（生产版） | 约31 KB  | 经过工具压缩或经过服务器开启Gzip压缩 主要应用于产品和项目 |

### 1.2 jQuery简单案例


```html
    <script src="script/jquery-1.12.2.js"></script>
    <script>
        $(document).ready(function(){
            alert('简单案例测试');
        });
    </script>
```

1. 引入jQuery的js文件
2. 文件初始化代码`$(function () {});`与原生的window.onload()代码类似，不过还是有区别的，对它们进行了简单对比。

|            | window.onload                                                | $(document).ready()                 |
| :--------: | ------------------------------------------------------------ | ----------------------------------- |
|  执行时机  | 必须等到页面中所有的文件都加载完毕才会执行(DOM结构、js文件、图片等) | 网页中的DOM结构绘制完毕后就可以执行 |
| 编写的个数 | 不能同时的编写多个，当有多个window.onload=function(){};时，只有最后的一个有作用 | 能同时编写多个，都可以执行          |
|  简化写法  | 无                                                           | `$(function(){、、、})`;            |

### 1.3 jQuery代码风格

虽然jQuery做到了行为和内容的分离，但jQuery代码本身也应该拥有良好的层次结构及规范，这样才能进一步改善代码的可读性和可维护性

（1）对于同一个对象不超过3个操作的，可以直接写成一行

（2）对于同一个对象的较多操作，建议每行写一个操作

（3）对于多个对象的少量操作，可以每个对象写一行，如果涉及子元素，可以考虑适当地缩进

（4）为代码添加注释

### 1.4 jQuery对象和DOM对象

jQuery对象就是通过jQuery包装DOM对象后产生的对象。jQuery对象是jQuery独有的。如果一个对象是jQuery对象，那么就可以使用jQuery里的方法。例如：

```html
$('#foo').html()  //获取id为foo的元素内的html代码。html()是jQuery的方法
```

这段代码等同于：

```html
document.getElementById('foo').innerHtml()
```

在jQuery对象中无法使用DOM对象的任何方法。例如`$("#id").innerHTML`和`$("#id").checked`之类的写法都是错误的，可以用`$("#id").html()`和`$("#id").attr("checked")`之类的jQuery方法来代替。同样，DOM对象也不能使用jQuery里的方法。例如`document.getElementById("id").html()`也会报错，只能用`document.getElementById("id").innerHTML`语句。

jQuery对象和DOM对象是可以相互转化的

- jQuery对象转化为DOM对象

```html
var $cr = $('#cr');  
// 第一种方法
var cr  = $cr[0];
// 第二种方法
var cr  = $cr.get(0);
```

- DOM对象转化为jQuery对象

```javascript
var cr = document.getElementById('cr');
// 转化为jQuery对象
var $cr = $(cr);
```

## 2. 选择器

jQuery中的选择器完全继承了CSS的风格。利用jQuery选择器，可以非常便捷和快速地找出特定的DOM元素，然后为它们添加相应的行为，而无需担心浏览器是否支持这一选择器。学会使用选择器是学习jQuery的基础，jQuery的行为规则都必须在获取到元素后才能生效。

需要注意的是，$('#tt')获取的永远是对象，即使网页上没有此元素。因此当要用jQuery检查某个元素在网页上是否存在时，不能使用以下代码:

```javascript
if($('tt')){
    //do something
}
```

应该根据元素的长度判断

```javascript
if($('tt').length>0){
    //do something
}
```

### 2.1 基本选择器

基本选择器是jQuery中最常用的选择器，也是最简单的选择器，它通过元素id、class和标签名等来查找DOM元素。在网页中，每个id名称只能使用一次，class允许重复使用。基本选择器的介绍说明如表2-1所示。

​                                                                 **2-1 基本选择器**

| 选择器  | 描述                     | 返回     | 示例                              |
| :-----: | ------------------------ | -------- | --------------------------------- |
|   #id   | 根据给定的id匹配一个元素 | 单个元素 | `$('#test')`选取id为test的元素    |
| .class  | 根据给定的类名匹配元素   | 集合元素 | `$('.test')`选取class为test的元素 |
| element | 根据给定元素名匹配元素   | 集合元素 | `$(p)`选取所有的`<p>`元素         |

### 2.2 层次选择器

如果想通过DOM元素之间的层次关系来获取特定元素，例如后代元素、子元素、相邻元素和同辈元素等那么层次选择器是一个非常好的选择。层次选择器的介绍说明如表2-2所示

|          选择器          |                         描述                         |   返回   |                        示例                         |
| :----------------------: | :--------------------------------------------------: | :------: | :-------------------------------------------------: |
| `$(ancestor descendent)` | 选取ancestor（祖先）里面的所有descendent（后代）元素 | 集合元素 |       `$(div span)`选择div里的所有的span标签        |
|    `$(parent->child)`    |      选取parent下的child（子元素)，不包括孙子代      | 集合元素 |      $(div-> span)`选择div里的所有的子span标签      |
|      `$(pre+next)`       |           选取紧接在pre元素后面的next元素            | 集合元素 | `$(.one+div)`选取class为one的下一个`<div>`同辈元素  |
|    `$(pre~siblings)`     |          选取pre元素之后的所有siblings元素           | 集合元素 | `$(#two~div)`选取id为two的元素后面的所有`<div>`同辈 |

可以使用`next()`方法来替代`pre+next`

|          |     选择器      |          方法           |
| -------- | :-------------: | :---------------------: |
| 等价关系 | `$('.one+div')` | `$('.one').next('div')` |

可以使用`nextAll()`方法来代替`$('prev〜siblings')`选择器

|          |      选择器       |            方法            |
| -------- | :---------------: | :------------------------: |
| 等价关系 | `$('.pre ~ div')` | `$('.pre').nextAll('div')` |

在此我将后面要讲解的`siblings()`方法拿出来与`$("#prev〜div")`选择器只能选择“prev”元素后面的同辈`<div>`元素。而`siblings()`方法与前后位置无关，只要是同辈节点就都能匹配。

### 2.3 过滤选择器

过滤选择器主要是通过特定的过滤规则来筛选出所需的DOM元素，过滤规则与CSS中的伪类选择器语法相同，即选择器都以一个冒号(:)开头。按照不同的过滤规则，过滤选择器可以分为基本过滤、内容过滤、可见性过滤、属性过滤、子元素过滤和表单对象属性过滤选择器。这里只对最为常用的过滤器进行介绍

#### 2.3.1 基本过滤选择器

| 选择器            | 描述                                      | 返回     | 示例                                                         |
| ----------------- | ----------------------------------------- | -------- | ------------------------------------------------------------ |
| `:first`          | 选取第一个元素                            | 单个元素 | `$('div:first')`:选取所有`<div>`中的第一个元素               |
| `:last`           | 选取最后一个元素                          | 单个元素 | `$('div:last')`:选取所有`<div>`中的最后一个元素              |
| `：not(selector)` | 去除于给定选择器匹配的元素                | 集合元素 | `$(input:not(myClass)`：选取classs不是myClass的`<input>`元素 |
| `:even`           | 选取索引值为偶数的元素（索引从0开始）     | 集合元素 | `$(input:even`：选取索引是偶数的`<input>`元素                |
| `:odd`            | 选取索引值为奇数的元素（索引从0开始）     | 集合元素 | `$(input:odd)`：选取索引是奇数的`<input>`元素                |
| `:eq(index)`      | 选取索引值等于index的元素（index从0开始） | 集合元素 | `$(input:eq(1))`：选取索引等于1的`<input>`元素               |
| `:gt(index)`      | 选取索引值大于index的元素（index从0开始） | 集合元素 | `$(input:gt(1))`：选取索引大于1的`<input>`元素               |
| `:lt(index)`      | 选取索引值小于index的元素（index从0开始） | 集合元素 | `$(input:lt(1))`：选取索引大于1的`<input>`元素               |

#### 2.3.2 内容过滤器

| 选择器            | 描述                             | 返回     | 示例                                                      |
| ----------------- | -------------------------------- | -------- | --------------------------------------------------------- |
| `:contains(text)` | 选取含有文本内容为“text”的元素   | 集合元素 | `$("div.contains('我')")`选择文本含有‘我’的`<div>`元素    |
| `:has(selector)`  | 选择含有选择器所匹配的元素的元素 | 集合元素 | `$(div:has(p))`选取包含`<p>`元素的`<div>`元素             |
| `:empty`          | 选择不包含子元素或者是文本的元素 | 集合元素 | `$(div:empty)`选取不包括子元素（文本元素）的`<div>`空元素 |

#### 2.3.3 可见性过滤选择器

可见性过滤选择器是根据元素的可见和不可见状态来选择相应的元素

| 选择器    | 描述               | 返回     | 示例                                                         |
| --------- | ------------------ | -------- | ------------------------------------------------------------ |
| `:hidden` | 选取所有不可见元素 | 集合元素 | `$(:hidden)`选取所有的不可见的元素。包括`<input type = 'hidden'/>,<div style="display:none">和<div style="visibility:hidden">`等元素。如果只想选取`<input>`元素，可以使用`$(input:hidden)` |
| `:visibe` | 选取所有可见元素   | 集合元素 | `$("div:visibe")`选取所有可见的`<div>`元素                   |

#### 2.3.4 属性过滤选择器

| 选择器               | 描述                        | 返回     | 示例                                                         |
| -------------------- | --------------------------- | -------- | ------------------------------------------------------------ |
| `[attribute]`        | 选取拥有此属性的元素        | 集合元素 | `$("div[id]")`选取拥有属性为id的`<div>`元素                  |
| `[attribute=value]`  | 选取属性值为value的元素     | 集合元素 | `$("div[title=test]")`选取拥有属性title的值为text的`<div>`元素 |
| `[attribute*=value]` | 选取属性值含有value的元素   | 集合元素 | `$("div[title*=test]")`选取拥有属性title含有值为test的`<div>`元素 |
| `[attribute^=value]` | 选取属性值以value开头的元素 | 集合元素 | `$("div[title^=test]")`选取拥有属性title值以test开头的`<div>`元素 |
| `[attribute$=value]` | 选取属性值以value结尾的元素 | 集合元素 | `$("div[title$=test]")`选取拥有属性title值以test结尾的`<div>`元素 |

#### 2.3.5 子元素选择器

子元素过滤选择器的过滤规则相对于其它的选择器稍微有些复杂，不过没关系，只要将元素的父元素和子元素区分清楚，那么使用起来也非常简单。另外还要注意它与普通的过滤选择器的区别。

| 选择器                                 | 描述                                                      | 返回     | 示例                                                         |
| -------------------------------------- | --------------------------------------------------------- | -------- | ------------------------------------------------------------ |
| `：nth-child(index/even/odd/equation)` | 选取父元素下的第index个子元素或者奇偶元素（index从1开始） | 集合元素 | `:eq(index)`只匹配一个元素，而`:nth-child(index)`为每一个父元素匹配子元素，并且index的值     从1开始的 |
| `：first-child`                        | 选取父元素的第一个元素                                    | 集合元素 | `:first`只返回单个元素，而`:first-child`选择符将为每一个父元素匹配子元素。例如`$(ul li:first-child)`选取每个`<ul>`中第一个`<li>`元素 |
| `：last-child`                         | 选取父元素的最后一个元素                                  | 集合元素 | `:last`只返回单个元素，而`:last-child`选择符将为每一个父元素匹配子元素。例如`$(ul li:first-child)`选取每个`<ul>`中最后一个`<li>`元素 |

`:nth-child()`选择器是很常用的子元素过滤选择器，详细功能如下。

（1）:nth-child(even)能选取每个父元素下的索引值是偶数的元素。

（2）:nth-child(odd)能选取每个父元素下的索引值是奇数的元素。

（3）:nth-child(2)能选取每个父元素下的索引值等于2的元素。

（4）:nth-child(3n)能选取每个父元素下的索引值是3的倍数的元素，（n从1开始）。

（5）:nth-child(3n+1)能选取每个父元素下的索引值是（3n+1）的元素。（n从1开始）

#### 2.3.6 表单对象属性过滤选择器

此选择器主要是对所选择的表单元素进行过滤，例如选择被选中的下拉框，多选框等元素。表单对象属性

| 选择器      | 描述                                 | 返回     | 示例                                                         |
| ----------- | ------------------------------------ | -------- | ------------------------------------------------------------ |
| `:enabled`  | 选取所有可用的元素                   | 集合元素 | `$(#form1 :enabled)`选择id为“form1”的表单内的所有可用元素    |
| `:disabled` | 选取所有不可用的元素                 | 集合元素 | `$(#form2 :disabled)`选择id为“form2”的表单内的所有不可用元素 |
| `:checked`  | 选取所有可用的元素（单选框，复选框） | 集合元素 | `$("input:checked")`选择所有被选中的`<input>`元素            |
| `:selected` | 选取所有被选中的选项元素             | 集合元素 | `$(select option:selected)`选取所有被选中项的元素            |
|             |                                      |          |                                                              |

### 2.4 表单选择器

为了使用户能够更加灵活地操作表单，jQuery中专门加入了表单选择器。利用这个选择器，能极其方便地获取到表单的某个或某类型的元素。

 

| 选择器      | 描述                                                | 返回     | 示例                                |
| ----------- | --------------------------------------------------- | -------- | ----------------------------------- |
| `:input`    | 选取所有的`<input>、<textarea>、<select>、<button>` | 集合元素 | `$(":input")`：选取所有的上述元素   |
| `:text`     | 选取所有的单行文本框                                | 集合元素 | `$(":text")`:选取所有的文本元素     |
| `:password` | 选取所有的密码框                                    | 集合元素 | `$(":password")`:选取所有的密码元素 |
| `:radio`    | 选取所有的单选框                                    | 集合元素 | `$(":radio")`:选取所有的单选元素    |
| `:checkbox` | 选取所有的多选框                                    | 集合元素 | `$(":checkbox")`选取所有的复选框    |
| `:submit`   | 选取提交按钮                                        | 集合元素 | `$(":submit")`选取所有的提交按钮    |
| `:image`    | 选取图像按钮                                        | 集合元素 | `$(":image")`选取所有的图像按钮     |

### 2.5 jQuery的遍历方法

另外的常用的遍历方法如下表所示：

| 方法       | 描述                                                         | 返回     | 示例                                                         |
| ---------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| siblings() | 返回被选元素的所有同胞元素                                   | 集合元素 | `$("h2").siblings("p");`返回属于` <h2> `的同胞元素的所有` <p> `元素： |
| children() | 返回被选元素的所有直接子元素                                 | 集合元素 | `$("div").children("p.1");`返回类名为 "1" 的所有` <p>` 元素，并且它们是` <div> `的直接子元素： |
| find()     | 返回被选元素的后代元素，一路向下直到最后一个后代             | 集合元素 | `$("div").find("span");`返回属于 `<div>` 后代的所有` <span>` 元素： |
| parent()   | 返回被选元素的直接父元素。该方法只会向上一级对 DOM 树进行遍历。 | 集合元素 | `$("span").parent();`返回每个` <span>` 元素的的直接父元素    |
| parents()  | 返回被选元素的所有祖先元素，它一路向上直到文档的根元素 `<html>` | 集合元素 | `$("span").parents();`返回所有`<span>` 元素的所有祖先        |
| first()    | 返回被选元素的首个元素                                       | 单个元素 | `$("div p").first();`选取首个 `<div> `元素内部的第一个 `<p>` 元素： |
| last()     | 返回被选元素的最后一个元素                                   | 单个元素 | `$("div p").last();`选择最后一个 `<div> `元素中的最后一个 `<p> `元素： |
| eq()       | 返回被选元素中带有指定索引号的元素                           | 单个元素 | `$("p").eq(1);`引号从 0 开始，因此首个元素的索引号是 0 而不是 1。下面的例子选取第二个 `<p> `元素（索引号 1）： |
| filter()   | filter() 方法允许您规定一个标准。不匹配这个标准的元素会被从集合中删除，匹配的元素会被返回 | 集合元素 | `$("p").filter(".url");`返回带有类名 "url" 的所有` <p>` 元素： |
| not()      | not() 方法返回不匹配标准的所有元素。                         | 集合元素 | `$("p").not(".url");`返回不带有类名 "url" 的所有 `<p> `元素： |



## 3. 样式操作

### 3.1获取和设置样式

HTML代码如下：

```html
<p class='myClass' title="选择你喜欢的水果">你最喜欢的水果是?</p>
```

在上面的代码中，class也是`<p>`元素的属性，因此获取class和设置class都可以使用attr()方法来完成。

例如使用attr()方法来获取<p>元素的class，jQuery代码如下：

```javascript
var p_class = $("p").attr("class");
```

也可以使用attr()方法来设置`<p>`元素的class，jQuery代码如下：

```javascript
$("p").attr("class","high");
```

运行代码后，上面的HTML代码将变为如下结构：

```html
<p class='high' title="选择你喜欢的水果">你最喜欢的水果是?</p>
```

上面的代码是将原来的`class（myClass）`替换为新的`class（high`）。如果此处需要的是“追加”效果，class属性变为“myClass high”，即`myClass`和`high`两种样式的叠加，那么我们可以使用==`addClass()`==方法。

### 3.2 追加样式

jQuery提供了专门的addClass()方法来追加样式。为了使例子更容易理解，首先在`<style>`标签里添加另一组样式：

```html
<style>
    .high {
        font-weight:700;
        color:red
    }
    .another {
        font-style:italic;
        color:'blue'
    }
</style>
```

然后在网页中添加一个“追加class类”的按钮，按钮的事件代码如下：

```javascript
$("p").addClass("another");
```

（1）如果给一个元素添加了多个class值，那么就相当于合并了它们的样式。

（2）如果有不同的class设定了同一样式属性，则后者覆盖前者。

### 3.3 移除样式

在上面的例子中，为`<p>`元素追加了another样式。此时`<p>`元素的HTML代码变为：

```html
<p class='high' title="选择你喜欢的水果">你最喜欢的水果是?</p>
```

如果用户单击某个按钮时，要删除class的某个值，那么可以使用与addClass()方法相反的removeClass()方法

来完成，它的作用是从匹配的元素中删除全部或者指定的class。

例如可以使用如下的jQuery代码来删除`<p>`元素中值为“high”的class:

```javascript
$("p").removeClass("high");
```

jQuery提供了更简单的方法。可以以空格的方式删除多个class名，jQuery代码如下：

```javascript
$("p").removeClass("high another");
```

另外，还可以利用removeClass()方法的一个特性来完成同样的效果。当它不带参数时，就会将class的值全部删除，jQuery代码如下：

```javascript
$("p").removeClass();
```

### 3.4 切换样式

jQuery还提供了一个toggleClass()方法控制样式上的重复切换。如果类名存在则删除它，如果类名不存在则添加它。例如对`<p>`元素进行toggleClass()方法操作。jQuery代码如下：

```javascript
$("p").toggleClass('another');  //重复的切换样式
```

### 3.5 判断是否含有某个样式

hasClass()可以用来判断元素中是否含有某个class，如果有，则返回true，否则返回false。

例如可以使用下面的代码来判断`<p>`元素中是否含有“another”的class：

```javascript
$("p").hasClass("another");
```

> 注意：这个方法是为了增强代码可读性而产生的。在jQuery内部实际上是调用了is()方法来完成这个功能的。该方法等价于如下代码：$("p").is(".another");

## 4.链式编程                      

### 4.1 链式编程要点

链式编程就是：多行代码合并成一行代码,前提要认清此行代码返回的是不是对象.是对象才能进行链式编程

```javascript
 .html(‘val’).text(‘val’).css()
```

链式编程，隐式迭代。

链式编程注意：

```html
$(‘div’).html().text()
```

这样是不对的，因为获取值时返回的是获取的字符串而不是对象本身所以不能链式编程。

### 4.2 链式编程end()方法返回到上一级

通过end()方法取消当前的jQuery对象，返回前面的jQuery对象。

这样当匹配某个按钮时，为其绑定事件处理函数，然后调用end()方法，则又返回前面一个jQuery对象，即按钮集合。

链式代码已经成为jQuery 非常流行的一个特点，在使用链条方式编写代码时，可能会用到eq()、filter()的jQuery方法改变链式方法的对象，但是借助jQuery的end() 方法又能够恢复或最初的jQuery对象，从而可以实现继续执行链式操作。注意，有几个jQuery的方法并不返回jQuery 对象，所以链式操作就不能继续下去，如get() 就不能像eq() 那样使用。

```javascript
$(‘ul.first’).find(’.foo’).css(‘background-color’, ‘red’)
.end().find(’.bar’).css(‘background-color’, ‘green’);
```

  

## 5. 动画

jQuery提供的一组网页中常见的动画效果，这些动画是标准的、有规律的效果；同时还提供给我们了自定义动画的功能。

### 5.1 显示动画

方式一：

```javascript
$("div").show();
```

解释：无参数，表示让指定的元素直接显示出来。其实这个方法的底层就是通过`display: block;`实现的。

方式二：

```javascript
    $("div").show(2000);
```

解释：通过控制元素的宽高、透明度、display属性，逐渐显示，2秒后显示完毕。

方式三：

```javascript
    $("div").show("slow");
```

参数可以是：

- slow 慢：600ms
- normal 正常：400ms
- fast 快：200ms

解释：和方式二类似，也是通过控制元素的宽高、透明度、display属性，逐渐显示。

方式四：

```
    //show(毫秒值，回调函数;
    $("div").show(5000,function () {
        alert("动画执行完毕！");
    });
```

解释：动画执行完后，立即执行回调函数。

**总结：**

上面的四种方式几乎一致：参数可以有两个，第一个是动画的执行时长，第二个是动画结束后执行的回调函数。

### 5.2 隐藏动画

方式参照上面的show()方法的方式。如下：

```javascript
    $(selector).hide();

    $(selector).hide(1000); 

    $(selector).hide("slow");

    $(selector).hide(1000, function(){});
```

**显示和隐藏的来回切换：**

显示和隐藏的来回切换采用的是toggle()方法：就是先执行show()，再执行hide()。

同样是四种方式：

```javascript
$(selector).toggle();
```

### 5.3 滑入和滑出

**1、滑入动画效果**：（类似于生活中的卷帘门）

```javascript
    $(selector).slideDown(speed, 回调函数);
```

解释：下拉动画，显示元素。

注意：省略参数或者传入不合法的字符串，那么则使用默认值：400毫秒（同样适用于fadeIn/slideDown/slideUp）

**2 滑出动画效果：**

```javascript
    $(selector).slideUp(speed, 回调函数);
```

解释：上拉动画，隐藏元素。

**3、滑入滑出切换动画效果：**

```javascript
    $(selector).slideToggle(speed, 回调函数);
```

参数解释同show()方法。

举例：

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <style>
        div {
            width: 300px;
            height: 300px;
            display: none;
            background-color: pink;
        }
    </style>

    <script src="jquery-1.11.1.js"></script>
    <script>
        $(function () {
            //点击按钮后产生动画
            $("button:eq(0)").click(function () {

                //滑入动画： slideDown(毫秒值，回调函数[显示完毕执行什么]);
                $("div").slideDown(2000, function () {
                    alert("动画执行完毕！");
                });
            })

            //滑出动画
            $("button:eq(1)").click(function () {

                //滑出动画：slideUp(毫秒值，回调函数[显示完毕后执行什么]);
                $("div").slideUp(2000, function () {
                    alert("动画执行完毕！");
                });
            })

            $("button:eq(2)").click(function () {
                //滑入滑出切换（同样有四种用法）
                $("div").slideToggle(1000);
            })

        })
    </script>
</head>
<body>
<button>滑入</button>
<button>滑出</button>
<button>切换</button>
<div></div>

</body>
</html>
```

![img](http://img.smyhvae.com/20180205_1420.gif)

### 5.4 淡入淡出动画

1、淡入动画效果：

```javascript
    $(selector).fadeIn(speed, callback);
```

作用：让元素以淡淡的进入视线的方式展示出来。

2、淡出动画效果：

```javascript
    $(selector).fadeOut(1000);
```

作用：让元素以渐渐消失的方式隐藏起来

3、淡入淡出切换动画效果：

```javascript
    $(selector).fadeToggle('fast', callback);
```

作用：通过改变透明度，切换匹配元素的显示或隐藏状态。

参数的含义同show()方法。

代码举例：

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <style>
        div {
            width: 300px;
            height: 300px;
            display: none;
            /*opacity: 1;*/
            background-color: pink;
        }
    </style>

    <script src="jquery-1.11.1.js"></script>
    <script>
        $(function () {
            //点击按钮后产生动画
            $("button:eq(0)").click(function () {
               //淡入动画用法1:   fadeIn();   不加参数
                $("div").fadeIn();
            })

            //滑出动画
            $("button:eq(1)").click(function () {
                //滑出动画用法2:   fadeOut(2000);   毫秒值
               $("div").fadeOut(2000);  //通过这个方法实现的：display: none;
               //通过控制  透明度和display
              
            })

            $("button:eq(2)").click(function () {
                //滑入滑出切换
                //同样有四种用法
                $("div").fadeToggle(1000);
            })

            $("button:eq(3)").click(function () {
                //改透明度
                //同样有四种用法
                $("div").fadeTo(1000, 0.5, function () {
                    alert(1);
                });
            })
        })
    </script>
</head>
<body>
<button>淡入</button>
<button>淡出</button>
<button>切换</button>
<button>改透明度为0.5</button>
<div></div>

</body>
</html>
```

### 5.5 自定义动画

```
    $(selector).animate({params}, speed, callback);
```

作用：执行一组CSS属性的自定义动画。

- 第一个参数表示：要执行动画的CSS属性（必选）
- 第二个参数表示：执行动画时长（可选）
- 第三个参数表示：动画执行完后，立即执行的回调函数（可选）

代码举例：

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <style>
        div {
            position: absolute;
            left: 20px;
            top: 30px;
            width: 100px;
            height: 100px;
            background-color: pink;
        }
    </style>
    <script src="jquery-1.11.1.js"></script>
    <script>
        jQuery(function () {
            $("button").click(function () {
                var json = {"width": 500, "height": 500, "left": 300, "top": 300, "border-radius": 100};
                var json2 = {
                    "width": 100,
                    "height": 100,
                    "left": 100,
                    "top": 100,
                    "border-radius": 100,
                    "background-color": "red"
                };

                //自定义动画
                $("div").animate(json, 1000, function () {
                    $("div").animate(json2, 1000, function () {
                        alert("动画执行完毕！");
                    });
                });

            })
        })
    </script>
</head>
<body>
<button>自定义动画</button>
<div></div>
</body>
</html>
```

### 5.6 停止动画

```
    $(selector).stop(true, false);
```

**里面的两个参数，有不同的含义。**

第一个参数：

- true：后续动画不执行。
- false：后续动画会执行。

第二个参数：

- true：立即执行完成当前动画。
- false：立即停止当前动画。

PS：参数如果都不写，默认两个都是false。实际工作中，直接写stop()用的多。

**效果演示：**

当第二个参数为true时，效果如下：

![img](http://img.smyhvae.com/20180205_1445.gif)

当第二个参数为false时，效果如下：

![img](http://img.smyhvae.com/20180205_1450.gif)

这个**后续动画**我们要好好理解，来看个例子。

**案例：鼠标悬停时，弹出下拉菜单（下拉时带动画）**

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <style type="text/css">
        * {
            margin: 0;
            padding: 0;
        }

        ul {
            list-style: none;
        }

        .wrap {
            width: 330px;
            height: 30px;
            margin: 100px auto 0;
            padding-left: 10px;
            background-color: pink;
        }

        .wrap li {
            background-color: green;
        }

        .wrap > ul > li {
            float: left;
            margin-right: 10px;
            position: relative;
        }

        .wrap a {
            display: block;
            height: 30px;
            width: 100px;
            text-decoration: none;
            color: #000;
            line-height: 30px;
            text-align: center;
        }

        .wrap li ul {
            position: absolute;
            top: 30px;
            display: none;
        }
    </style>
    <script src="jquery-1.11.1.js"></script>
    <script>
        //入口函数
        $(document).ready(function () {
            //需求：鼠标放入一级li中，让他里面的ul显示。移开隐藏。
            var jqli = $(".wrap>ul>li");

            //绑定事件
            jqli.mouseenter(function () {
                $(this).children("ul").stop().slideDown(1000);
            });

            //绑定事件(移开隐藏)
            jqli.mouseleave(function () {
                $(this).children("ul").stop().slideUp(1000);
            });
        });
    </script>

</head>
<body>
<div class="wrap">
    <ul>
        <li>
            <a href="javascript:void(0);">一级菜单1</a>
            <ul>
                <li><a href="javascript:void(0);">二级菜单1</a></li>
                <li><a href="javascript:void(0);">二级菜单2</a></li>
                <li><a href="javascript:void(0);">二级菜单3</a></li>
            </ul>
        </li>
        <li>
            <a href="javascript:void(0);">一级菜单1</a>
            <ul>
                <li><a href="javascript:void(0);">二级菜单1</a></li>
                <li><a href="javascript:void(0);">二级菜单2</a></li>
                <li><a href="javascript:void(0);">二级菜单3</a></li>
            </ul>
        </li>
        <li>
            <a href="javascript:void(0);">一级菜单1</a>
            <ul>
                <li><a href="javascript:void(0);">二级菜单1</a></li>
                <li><a href="javascript:void(0);">二级菜单2</a></li>
                <li><a href="javascript:void(0);">二级菜单3</a></li>
            </ul>
        </li>
    </ul>
</div>
</body>
</html>
```

效果如下：

![img](http://img.smyhvae.com/20180205_1500.gif)

> 当鼠标第一次进入时，会触发` $(this).children("ul").stop().slideDown(1000);`，此时stop()会立即的停止当前的动画，由于当前没有动画，所以会执行后续的`slideDown(1000)`，此时，如果鼠标在1秒移除，会触发`$(this).children("ul").stop().slideUp(1000);`，此时stop()方法会停止未执行完的`slideDown(1000)`(滑入动画)，进而执行后续的`slideUp(1000)`(滑出动画）；如果不加stop()方法会先让滑入执行完再执行滑出动画。

上方代码中，关键的地方在于，用了stop函数，再执行动画前，先停掉之前的动画。

如果去掉stop()函数，效果如下：（不是我们期望的效果）

![img](http://img.smyhvae.com/20180205_1505.gif)

stop方法的总结

> 当调用stop()方法后，队列里面的下一个动画将会立即开始。
> 但是，如果参数clearQueue被设置为true，那么队列面剩余的动画就被删除了，并且永远也不会执行。如果
>
> 参数jumpToEnd被设置为true，那么当前动画会停止，但是参与动画的每一个CSS属性将被立即设置为它们的目标值。比如：slideUp()方法，那么元素会立即隐藏掉。如果存在回调函数，那么回调函数也会立即执行。

注意：如果元素动画还没有执行完，此时调用sotp()方法，那么动画将会停止。并且动画没有执行完成，那么回调函数也不会被执行。

## 6. DOM节点的相关操作

### 6.1 创建元素

DOM中可以动态创建元素:

- document.write() : 会把页面中的所有的页面全部覆盖
- element.innerHTML : 
- document.createElement() <推荐的方式>

jQuery中同样可以创建元素标签:

- `$(标签代码)`
- `对象.html("标签代码")`

例如要创建两个`<li>`元素节点，并且要把它们作为`<ul>`元素节点的子节点添加到DOM节点树上。完成这个任务需要两个步骤。

（1）创建两个`<li>`新元素。

```javascript
var $li_1 = $("<li></li>");
var $li_2 = $("<li></li>");
```

（2）将这两个新元素插入文档中。

```javascript
$("ul").append($li_1)
```

> 注意：
>
> （1）动态创建的新元素节点不会被自动添加到文档中，而是需要使用其他方法将其插入文档中。
>
> （2）当创建单个元素时，要注意闭合标签和使用标准的XHTML格式。例如创建一个`<p>`元素，可以用`$("</p>")`或者`$("<p></p>")`，但不要使用`$("<p>")`或者大写的`$("<P/>")`。

### 6.2 插入节点

动态创建HTML元素并没有实际用处，还需要将新创建的元素插入文档中。将新创建的节点插入文档最简单的办法是，让它成为这个文档的某个节点的子节点。前面使用了一个插入节点的方法append()，它会在元素内部追加新创建的内容。

将新创建的节点插入某个文档的方法并非只有一种，在jQuery中还提供了其他几种插入节点的方法，如表所示。读者可以根据实际需求灵活地做出多种选择。

![mark](http://man.hhaxmm.cn/blog/20190415/U5oe8WRdVSBX.png)

这些插入节点的方法不仅能将新创建的DOM元素插入到文档中，也能对原有的DOM元素进行移动。

简单举例

```html
<body>
    <p title="选择你喜欢的水果">选择你喜欢的水果</p>
    <input type="button" value="元素移动" id="btn1">
    <input type="button" value="插入元素" id="btn2">
    <input type="button" value="元素移动" id="btn">
    <ul>
        <li title="苹果">苹果</li>
        <li title="橘子">橘子</li>
        <li title="梨子">梨子</li>
    </ul>
</body>
```

对元素进行插入和移动操作

```html
 <script>
     // 移动元素
        $(function () {
            $('#btn').click(function () {
                var $one_li = $("li:eq(1)");
                var $two_li = $("li:eq(2)");
                console.log('置顶');
                $two_li.insertBefore($one_li);
            });
        });
     
      // 插入元素
        $(function(){
            var $li_1 = $("<li title = '香蕉'>香蕉</li>");
            var $li_2 = $("<li title = '雪梨'>雪梨</li>");
            var $li_3 = $("<li title = '其他'>其他</li>");
            
            var $parent = $("ul");
            var $two_li = $("ul li:eq(1)");
            
            $('#btn2').click(function(){
                $parent.append($li_1);
                $parent.prepend($li_2);
                console.log($two_li.html());
                console.log($li_3.html());
                $two_li.after($li_3);
            });

        });
</script>
```

### 6.3 删除节点

如果文档中某一个元素多余，那么应将其删除。jQuery提供了三种删除节点的方法，即

- remove()，
- detach()
- empty()

**1．remove()方法**

作用是从DOM中删除所有匹配的元素，传入的参数用于根据jQuery表达式来筛选元素。

例如删除中`<ul>`节点中的第2个`<li>`元素节点，jQuery代码如下：

```javascript
$("ul li:eq(1)").remove();
```

当某个节点用remove()方法删除后，该节点所包含的所有后代节点将同时被删除。这个方法的返回值是一个指向已被删除的节点的引用，因此可以在以后再使用这些元素。下面的jQuery代码说明元素用remove()方法删除后，还是可以继续使用的。

```javascript
var $li_2 = $("ul li:eq(1)").remove();
$li_2.appendTo($('ul'));
```

另外remove()方法也可以通过传递参数来选择性地删除元素，jQuery代码如下：

```javascript
$("ul li").remove("li[title!='橘子']");
```

**2．detach()方法**

detach()和remove()一样，也是从DOM中去掉所有匹配的元素。但需要注意的是，这个方法不会把匹配的元素从jQuery对象中删除，因而可以在将来再使用这些匹配的元素。与remove()不同的是，所有绑定的事件、附加的数据等都会保留下来。

```javascript
            $("ul li").click(function () {
                alert($(this).html());
            });
            // 删除元素
            var $li = $("ul li:eq(1)").detach();
            // 重新的添加，事件任然会绑定，不会消失,remove（）方法会消失
            $li.appendTo($("ul"));
        });
```

**3．empty()方法**

 严格来讲，empty()方法并不是删除节点，而是清空节点，它能清空元素中的所有后代节点。jQuery代码如下：

```javascript
 // 清空元素的内容，注意是清空内容
 $('ul li:eq(1)').empty();
```

**案例演示**

详细的代码在我的githube上显示[jQuery增加删除节点](<https://github.com/huanghe1993/jQuery-/blob/master/jQuery%E4%BD%93%E9%AA%8C/jQuery002%E5%85%83%E7%B4%A0%E7%9A%84%E7%A7%BB%E5%8A%A8.html>)

### 6.4 复制节点

继续沿用之前水果的例子，如果单击`<li>`元素后需要再复制一个`<li>` 元素，可以使用clone()方法来完成，jQuery代码如下：

```javascript
$(function(){
            $("ul li").click(function(){
                $(this).clone().appendTo($("ul"));
            })
        });
```

复制节点后，被复制的新元素并不具有任何行为。如果需要新元素也具有复制功能（本例中是单击事件），可以使用如下jQuery代码：

```javascript
$(function(){
            $("ul li").click(function(){
                $(this).clone(true).appendTo($("ul"));
            })
        });
```

在clone()方法中传递了一个参数true，它的含义是复制元素的同时复制元素中所绑定的事件。因此该元素的副本也同样具有复制功能（本例中是单击事件）。

### 6.5 替换节点

如果要替换某个节点，jQuery提供了相应的方法，即`replaceWith()`和`replaceAll()`。`replaceWith()`方法的作用是将所有匹配的元素都替换成指定的HTML或者DOM元素。例如要将网页中“`<p title="选择你最喜欢的水果.">你最喜欢的水果是？</p>”替换成“<strong>你最不喜欢的水果是？</strong>”，`可以使用如下jQuery代码：

```javascript
        // 替换节点 replaceWith
        $(function(){
            $('#btn1').click(function (e) { 
                e.preventDefault();
                $("p").replaceWith("<strong>你最喜欢的水果是？</strong><br>");
            });
        });
```

也可以使用jQuery中另一个方法replaceAll()来实现，该方法与replaceWith()方法的作用相同，只是颠倒了replaceWith()操作，可以使用如下jQuery代码实现同样的功能：

```javascript
$("<strong>你最喜欢的水果是？</strong><br>").replaceAll($("p"));
```

> 注意：如果在替换之前，已经为元素绑定事件，替换后原先绑定的事件将会与被替换的元素一起消失，需要在新元素上重新绑定事件。

### 6.6 属性操作

在jQuery中，用attr()方法来获取和设置元素属性，removeAttr()方法来删除元素属性。

**1．获取属性和设置属性**

 如果要获取`<p>`元素的属性title，那么只需要给attr()方法传递一个参数，即属性名称。jQuery代码如下：

```javascript
 var p_txt = $('p').attr('title');
console.log(p_txt);
```

如果要设置`<p>`元素的属性title的值，也可以使用同一个方法，不同的是，需要传递两个参数即属性名称和对应的值。jQuery代码如下：

```javascript
$('p').attr('title','your title!');
```

如果需要一次性为同一个元素设置多个属性，可以使用下面的代码来实现：

```javascript
// 使用json串的格式
$("p").attr({"title":"your title","name":'test'});
```

> 注意：
>
> jQuery中的很多方法都是同一个函数实现获取（getter）和设置（setter）的，例如上面的attr()方法，既能设置元素属性的值，也能获取元素属性的值。类似的还有html()、text()、height()、width()、val()和css()等方法。

**2．删除属性** 

在某些情况下，需要删除文档中某个元素的特定属性，可以使用removeAttr()方法来完成该任务。

如果需要删除`<p>`元素的title属性，可以使用下面的代码实现：

```javascript
 $('p').removeAttr("title");
```

> 注意：jQuery1.6中新增了prop()和removeProp()，分别用来获取在匹配的元素集中的第一个元素的属性值和为匹配的元素删除设置的属性。

### 6.7 样式操作

**1．获取样式和设置样式** 

HTML代码如下：

```javascript
<p class="myClass" title="选择你喜欢的水果">选择你喜欢的水果</p>
```

在上面的代码中，class也是`<p>`元素的属性，因此获取class和设置class都可以使用attr()方法来完成。

例如使用attr()方法来获取`<p>`元素的class，jQuery代码如下：

```javascript
var p_class = $("p").attr("class");  //获取p元素的class
```

也可以使用attr()方法来设置`<p>`元素的class，jQuery代码如下：

```javascript
$("p").attr("class","high"); //设置<p>元素的class为high
```

上面的代码是将原来的class（myClass）替换为新的class（high）。如果此处需要的是“追加”效果，class属性变为“myClass high”，即myClass和high两种样式的叠加，那么我们可以使用addClass()方法。

>  其他的细节可以查看样式的操作小节

### 6.7 设置和获取html、文本和值

**1．html()方法**

 此方法类似于JavaScript中的innerHTML属性，可以用来读取或者设置某个元素中的HTML内容。

 ```javascript
 var p_html = $("p").html();
 ```

如果需要设置某元素的HTML代码，那么也可以使用该方法，不过需要为它传递一个参数。例如要设置<p>元素的HTML代码，可以使用如下代码：

```javascript
 // 设置p的元素html代码
$("p").html("<strong>选择你喜欢的水果?</strong>")
```

**2．text()方法**

 此方法类似于JavaScript中的innerText属性，可以用来读取或者设置某个元素中的文本内容。继续使用以上的HTML代码

```html
<p class="myClass" title="选择你喜欢的水果"><strong>选择你喜欢的水果</strong></p>
```

用text()方法对`<p>`元素进行操作：

```javascript
 var p_text = $("p").text();
```

与html()方法一样，如果需要为某元素设置文本内容，那么也需要传递一个参数。例如对`<p>`元素设置文本内容，代码如下：

```javascript
 $("p").text("text方法修改后的值");
```

> 注意：
>
> （1）JavaScript中的innerText属性并不能在Firefox浏览器下运行，而jQuery的text()方法支持所有的浏览器。
> （2）text()方法对HTML文档和XML文档都有效。

**3．val()方法**

 此方法类似于JavaScript中的value属性，可以用来设置和获取元素的值。无论元素是文本框，下拉列表还是单选框，它都可以返回元素的值。如果元素为多选，则返回一个包含所有选择的值的数组。

简单的案例说明：获得焦点、失去焦点的情况

```html
 <script>
        $(function(){
            $("#address").focus(function(){
                var text_value = $(this).val();
                console.log(text_value);
                // 还可以用this.defaultValue
                if (text_value =="请输入邮箱地址"){
                    $(this).val("");
                }
            });
            $("#address").blur(function(){
                var text_value = $(this).val();
                if (text_value ==""){
                    $(this).val("请输入邮箱地址");
                }
            });
        });
    </script>
</head>
<body>
    <input type="text" name="" id="address" value="请输入邮箱地址"><br><br>
    <input type="text" name="" id="password" value="请输入邮箱密码"><br><br>
    <input type="button" value="登录">
```

### 6.8 CSS操作

**1：CSS样式颜色**

CSS-DOM技术简单来说就是读取和设置style对象的各种属性。style属性很有用，但最大不足是无法通过它来提取到通过外部CSS设置的样式信息，然而在jQuery中，这些都是非常的简单。

 可以直接利用css()方法获取元素的样式属性，jQuery代码如下：

```javascript
$("p").css("color");  //获取p元素的样式的颜色属性
```

无论color属性是外部CSS导入，还是直接拼接在HTML元素里（内联），css()方法都可以获取到属性style里的其他属性的值。

也可以直接利用css()方法设置某个元素的单个样式，例如：

```javascript
$("p").css("color","red");  //获取p元素的样式的颜色属性
```

与attr()方法一样，css()方法也可以同时设置多个样式属性，代码如下：

```javascript
$("p").css({"color":"red","fontSize":"30px"}); 
```

> 注意：
>
> （1）如果值是数字，将会被自动转化为像素值。
>
> （2）在css()方法中，如果属性中带有“-”符号，例如font-size和background-color属性，如果在设置这些属性的值的时候不带引号，那么就要用驼峰式写法，例如：
>
> `$("p").css({ fontSize : "30px"，backgroundColor : "#888888" })`
>
> 如果带上了引号，既可以写成“font-size”，也可以写成“fontSize”。
>
>  总之建议大家加上引号，养成良好的习惯。

对透明度的设置，可以直接使用opacity属性，jQuery已经处理好了兼容性的问题，如下代码所示，将<p>元素的透明度设置为半透明：

```javascript
$("p").css("opacity","0.5")
```

如果需要获取某个元素的height属性，则可以通过如下jQuery代码实现：

```javascript
$(element).css("height")
```

**2：宽度和高度**

在jQuery中还有另外一种方法也可以获取元素的高度，即height()。它的作用是取得匹配元素当前计算的高度值（px）。jQuery代码如下：

```javascript
$("p").height() //获取<p>元素的高度值
```

height()方法也能用来设置元素的高度，如果传递的值是一个数字，则默认单位为px。如果要用其他单位（例如em），则必须传递一个字符串。jQuery代码如下：

```javascript
$("p").height(100) //设置p元素的高度为100px
```

> 注意：
>
> （1）在jQuery1.2版本以后的height()方法可以用来获取window和document的高度。
> （2）两者的区别是：css()方法获取的高度值与样式的设置有关，可能会得到"auto"，也可能得到"10px"之类的字符串；而height()方法获取的高度值则是元素在页面中的实际高度，与样式的设置无关，并且不带单位。

与height()方法对应的还有一个width()方法，它可以取得匹配元素的宽度值（px）。

```javascript
$("p").width() //获取p元素的宽度
```

同样，width()方法也能用来设置元素的宽度。

```javascript
$("p").with("400px")  //设置p元素的宽度
```

**3：位置，偏移量**

- **offset()方法**

它的作用是获取元素在当前视窗的相对偏移，其中返回的对象包含两个属性，即top和left，它只对可见元素有效。例如用它来获取`<p>`元素的的偏移量，jQuery代码如下：

```javascript
var offset = $("p").offset(); //获取元素的offset
var left = offset.left; // 获取左偏移量
var top = offset.top //获取上偏移量
```

- **position()方法**

它的作用是获取元素相对于最近的一个position样式属性设置为relative或者absolute的祖父节点的相对偏移，与offset()一样，它返回的对象也包括两个属性，即top和left。jQuery代码如下：

```javascript
var position = $("p").offset(); //获取元素的position
var left = position.left; // 获取左偏移量
var top = position.top //获取上偏移量
```

- **scrollTop()方法和scrollLeft()方法**

这两个方法的作用分别是获取元素的滚动条距顶端的距离和距左侧的距离。例如使用下面的代码获取`<p>`元素的滚动条距离：

```javascript
var $p = $("p"); //获取元素
var top = $p.scrollTop();   // 获取上偏移量
var left = $p.scrollLeft(); //获取左偏移量
```

另外，可以为这两个方法指定一个参数，控制元素的滚动条滚动到指定位置。例如使用如下代码控制元素内的滚动条滚动到距顶端300和距左侧300的位置：

```javascript
$("textarea").scrollTop(300)  //元素的垂直滚动条到指定的地方
$("textarea").scrollLeft(300) //元素的水平滚动条到指定的地方
```

## 7. jQuery事件

### 7.1 事件的绑定

在文档装载完成后，如果打算为元素绑定事件来完成某些操作，则可以使用bind()方法来对匹配元素进行特定事件的绑定，bind()方法的调用格式为：

```javascript
// bind()方法：第一个参数是事件的名字  第二个参数是事件处理函数
```





## 8. jQuery插件



## 参考文章

> 《锋利的jQuery》