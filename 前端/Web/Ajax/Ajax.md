## Ajax

[TOC]

##  1. 介绍

### 1.1 Ajax介绍

Ajax的全称是Asynchronous JavaScript and XML 中文名称定义为异步的JavaScript和XML。
Ajax是Web2.0技术的核心
由多种技术集合而成，使用Ajax技术不必刷新整个页面，只需对页面的局部进行更新，可以节省网络带宽，提高页面的加载速度，从而缩短用户等待时间，改善用户体验

我们传统的web应用，当我们提交一个表单请求给服务器，服务器接收到请求之后，返回一个新的页面给浏览器，这种做法浪费了很多带宽，因为我们发送请求之前和获得的新页面两者中很多的html代码是相同的，由于每次用户的交互都需要向服务器发送请求，应用的访问时间取决于服务器的返回时间。而我们使用Ajax就不同了，Ajax只取回一些必须的数据，它使用SOAP、XML或者支持json 的Web Service接口，我们在客户端利用JavaScript处理来自服务器的响应，这样客户端和服务器之间的数据交互就减少了，然后用户请求就得到了加速。

Ajax是一种互联网网页技术；传统网页一般需要重载整个页面来完成刷新，因此在刷新页面时浏览器会出现短暂空白的情况。ajax技术可以在不重新加载整个网页的情况下，对网页的某部分进行更新，也不会出现浏览器空白的情况。

> 通俗的解释：ajax通俗点说就是不在网页刷新的前提下进行内容的更新。就拿页面点赞，当你点赞的时候，网页并没有刷新，但是数据发生了变化。再比如网页的加载更多内容，使用Ajax页面不需要重新加载，不刷新页面。

### 1.2 Web服务器的介绍

> tomcat 与 nginx，apache的区别是什么？
>
>  这三者都是web server，那他们各自有什么特点呢？他们之间的区别是什么呢？
>
> nginx 和 tomcat在性能上面有何异同。
>
> tomcat用在java后台程序上，java后台程序难道不能用apache和nginx吗？

| 服务器 | Apache                | Nginx                 | Tomcat                       |
| :----- | :-------------------- | :-------------------- | :--------------------------- |
| 类型   | Http服务器HTTP Server | Http服务器HTTP Server | 应用服务器Application Server |
| 资源   | 静态资源              | 静态资源              | 动态资源                     |

Tomcat等应用服务器有个统一的称呼叫：Web容器(**Web Container**)。类似的Web容器还有Jetty, Resin等。（比如：tomcat它可以处理Servlet、JSP、JSTL等，同时又能处理这些静态资源，像HTML、CSS等)，但是不如专业的 HTTP Server 那么强大，所以应用服务器往往是运行在 HTTP Server 的背后，执行应用，将动态的内容转化为静态的内容之后，通过 HTTP Server 分发到客户端。

Apache, IIS, Nginx等叫做Web服务器(**Web Server**)。这类兄弟，他们**只**能处理HTTP协议，例如我们在请求HTML,图片等这些静态资源的时候，可以通过Web服务器来完成。**但如果请求的是JSP,PHP等，这些时候，Web服务器就力不从心，只能将请求转给合适的人去处理**，如果请求JSP或Servlet，那这个合适的人就是上面的Web容器。

**两者的区别就是Tomcat不仅可以用于处理动态资源，处理Servlet、JSP，也能处理静态资源，而Apache只能处理静态资源**

这个时候，你一定会说，那Tomcat这小子能都干了，还要Apache干嘛呢？

> 哎，这小子当时学的挺多，也都挺好，但处理静态资源这事，没学特别精。就像说裁缝可能也自己会做饭，但总归不如厨师专业。

**所以处理静态资源的时候，可以在Tomcat前面配置一个Apache这类的Web Server,不仅可以高效处理静态资源，还可以把这些常用的静态资源缓存起来，甚至还能完成负载均衡的目的(当然，这还需要后面Web Container做集群)**。

nginx和apache的区别

**1、**nginx相对于apache的优点

**2、**作为 Web 服务器：相比 Apache，Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率，这点使 Nginx 尤其受到虚拟主机提供商的欢迎。

 **3、**Nginx 配置简洁, Apache 复杂 ，Nginx 静态处理性能比 Apache 高 3倍以上 ，Apache 对 PHP 支持比较简单，Nginx 需要配合其他后端用 ，Apache 的组件比 Nginx 多 ，现在 Nginx 才是 Web 服务器的首选

 **4、**最核心的区别在于apache是同步多进程模型，一个连接对应一个进程；nginx是异步的，多个连接（万级别）

apache和PHP的关系

> Apache服务器本身不能处理php动态语言脚本文件，就寻找并委托**PHP应用服务器**来处理（服务器端事先得安装PHP应用服务器），Apache服务器将用户请求访问的php文件（如index.php）文件交给PHP应用服务器。
>
> PHP应用服务器接收php文件（如index.php），打开并解释php文件，最终翻译成html静态代码，再将html静态代码交还给Apache服务器，Apache服务器将接收到的html静态代码输出到客户端浏览器（即用户）。

## 2. PHP简单教程

### 2.1 PHP运行配置

安装安装wampserver,我安装的是2.5 版本，安装步骤自行百度。

### 2.1 变量

```php
 	// php变量
    // 字符串的拼接为 .
    $num = 123;
    echo '<div>编号为：'.$num."</div>";
    // 单引号 变量为普通的字符串
    // 双引号解析变量
    echo "<div>编号为: $num</div>";
```

### 2.2 数组

```php
<?php
    // 数组
    $arr = array(1,2,3,4,5);
    print_r($arr);
	// 数组访问
	echo $arr[0]
        
    // 自定义键
    $arr1 = array("username"=>"zhangsan","sex"=>"male");
    print_r($arr1);
	echo $arr1["username"];

	$arr2 = array();
	$arr2[0] = array(1,2,3);
	$arr2[1] = array(4,5,6);
	$arr2[2] = array(7,8,9);	
?>
```

### 2.3 函数

PHP中函数使不会区分大小写的，但是javascript中是会区分大小写的

```php
<?php
    function foo($info){
        return $info;
    }
    $ret = foo('hi');
    echo $ret;
?>
```

系统函数

```php
<?php
    $arr = array("a"=>"111","b"=>"222","c"=>"333");
	// 转化为json格式
    $ret = json_encode($arr); //{"a":"111","b":"222","c":"333"}
    echo $ret;
?>
```

### 2.4 Get和Post请求

- $_GET
- $_POST

```php
<?php
    //url=http://localhost/page.php?name=huanghe
    $f = $_GET['name'];  //得到url地址中传递的参数
    echo $f;
?>
```

POST请求

```php
 	$uname = $_POST['username'];
    $pw = $_POST['password'];

    if ($uname == 'admin' && $pw == '123') {
        echo '登录成功';
    }
```

##  3. 原生Ajax

### 3.1 原生Ajax实现页面局部更新

 ```javascript
<script>
        window.onload = function () {
            var btn = document.getElementById("btn");
            btn.onclick = function () {

                var username = document.getElementById("username").value;
                var password = document.getElementById("password").value;
                //使用ajax发送请求
                // 1：使用XMLHttpRequest对象
                var xhr = new XMLHttpRequest();
                // 2:准备发送
                xhr.open('get','./01check.php?username='+username+'&password='+password,'true')
                // 3：执行发送 ，POST请求参数在这里传递，不需要转码
                xhr.send(null); // get请求需要添加null参数
                // 4：指定回调函数
                xhr.onreadystatechange = function () {
                    // 服务器端数据回来
                    if (xhr.readyState == 4) {
                        if (xhr.status == 200) {
                            var data = xhr.responseText;
                            var info = document.getElementById("info");
                            if (data == 1) {
                                info.innerHTML = "登录成功";
                            }else{
                                info.innerHTML = "用户名或者是密码错误";
                            }
                        }
                    } 
                }
            }
        }
</script>
 ```

- 创建XMLHttpRequest对象的请求参数

1、Get或者是Post请求；

2、URL地址;   encodeURI（）：可以对请求的参数进行编码

3、true是异步，false是同步，默认是true;

```javascript
<script>
        window.onload = function () {
            var btn = document.getElementById("btn");
            btn.onclick = function () {

                var username = document.getElementById("username").value;
                var password = document.getElementById("password").value;
                //使用ajax发送请求
                // 1：使用XMLHttpRequest对象
                var xhr = new XMLHttpRequest();
                // readyState =0
                console.log(xhr.readyState+"-----1------");
                // 2:准备发送
                var parma = username='+username+'&password='+password;
                xhr.open('post','./01check.php?,'true')
                // 3：执行发送 ，POST请求参数在这里传递，不需要转码
                xhr.setRequestHeader("Content-Type","application/x-www-form-urllenconded");
                xhr.send(parma); 
                // readyState =1
                console.log(xhr.readyState+"-----2------");
                // 4：指定回调函数，调用的条件是readyState发生变化（不包括从0-1）
                xhr.onreadystatechange = function () {
              		// readyState =2,3,4
                    console.log(xhr.readyState+"-----3------");
                    // 服务器端数据回来
                    if (xhr.readyState == 4) {
                        if (xhr.status == 200) {
                            //表示数据是否正常
                            var data = xhr.responseText;
                            var info = document.getElementById("info");
                            if (data == 1) {
                                info.innerHTML = "登录成功";
                            }else{
                                info.innerHTML = "用户名或者是密码错误";
                            }
                        }
                    } 
                }
            }
        }
</script>
```

- readyState =0：表示xhr对象创建完成
- readyState =1：表示已经发生了请求
- readyState =2：浏览器收到了服务器传递过来数据，还为解析
- readyState =3：正在解析
- readyState =4：解析完成，可以使用

返回的数据格式可以是XML：`xhr.responseXML;`

### 3.2 Ajax返回的JSON数据

JSON与普通的JS对象的区别：

- JSON数据没有变量
- JSON结尾数据的结尾没有分号
- JSON数据中的键必须使用双引号包括，js可以加双引号，也可以是其他的形式

```json
{
    "name":"张三",
    "age":12,
    "lover":["coding","palaying"],
	"friend":{
        "height":'180cm',
        "weight":"75kg"，  
    }
}
```

JS的JSON文件解析：

```javascript
var d = JSON.parse(data);  //字符串转化为对象
var str = JSON.stringify(d); //对象转化为字符串
//访问JSON对象
console.log(d.name);
console.log(d.age);
console.log(d.friend.height);
```

### 3.3 Ajax异步与同步

![mark](http://man.hhaxmm.cn/blog/20190419/GA6oihiVnBvp.png)

```javascript
console.log(1);
setTimeout(function(){
    console.log(2);
},10)
console.log(3);
```

输出的结果是1、3、2

> JS的事件处理机制：单线程+事件队列（事件队列中的任务的条件：1、主线程空闲，2：任务满足触发条件（延时事件达到，满足特定事件的触发条件，ajax的回调函数（服务器有数据回应）））

## 4. jQuery的Ajax的封装

```javascript
    <script>
        $(function () {
            $("#btn").click(function () {
                var username = $('#username').val();
                var password = $('#password').val();
                var data1 = "username="+username+"&password="+password;
                console.log(data1);
                $.ajax({
                    type: "get",
                    url: "01check.php",
                    data: data1,
                    // 或者是 data:{username:username,password:password},
                    dataType: "json",//支持xml,text,json,jsonp
                    success: function (response) {
                        if (response == 1) {
                            info.innerHTML = "登录成功";
                        } else {
                            info.innerHTML = "用户名或者是密码错误";
                        }
                    }
                });
            });
        });
    </script>
```

- data可以使用`{username:username,password:password}`

内部代码的实现

```javascript
/*
 * @Description: 自己封装的ajax
 * @Author: river
 * @LastEditors: huanghe
 * @Date: 2019-04-20 10:49:16
 * @LastEditTime: 2019-04-20 11:04:49
 */
function ajax(obj) {
    
    var defaults = {
        type:'get',
        data:{},
        url:'#',
        dataType:'text',
        async:true,
        success:function (data) {console.log(data);}
    }

    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            defaults[key] = obj[key];
        }
    }

    var xhr = null;
    if (window.XMLHttpRequest) {
        xhr = new XMLHttpRequest();
    } else {
        xhr = new ActiveXObject('Microsoft.XMLHTTP');
    }

    var param = '';
    for (var  key in obj.data) {
        param +=key+'='+obj.data[key]+'&';
    }
    if (param) {
        param = param.substring(0,param.length-1);
    }

    // get请求的参数是封装在get的url里面的
    if (defaults.type == 'get') {
        defaults.url +='?'+encodeURI(param);
    }

    xhr.open(defaults.type,defaults.url,defaults.async);

    var data = null;

    // post 请求的数据是封装在data里面的
    if (defaults.type == 'post') {
        data = param;
        xhr.setRequestHeader("Content-Type","application/x-www-form-urllenconded");
    }
    xhr.send(data);

    xhr.onreadystatechange = function () {
        // 服务器端数据回来
        if (xhr.readyState == 4) {
            if (xhr.status == 200) {
                var data = xhr.responseText;
                //判断是否为JSON
                if (defaults.dataType == 'json') {
                    data = JSON.parse(data);
                }
                defaults.success(data);
            }
        } 
    }
}
```

## 5. Ajax跨域

- 同源策略

1. 同源策略是浏览器的一种安全策略，所谓的同源值的是请求URL地址中的协议、域名、端口都相同，只要其中之一不相同就是跨域
2. 同源策略主要是保证浏览器的安全性
3. 在同源策略下，浏览器不允许Ajax跨域获取服务器的数据

跨域的解决方案：

1. JSONP
2. document.domain+iframe
3. location.hash+iframe
4. window.postMessage
5. flash等第三方插件



### 5.1 JSONP原理分析

- 静态script标签的src属性进行跨域请求

```java
<script src="http://tom.com/data.php"></script>
```

> 这种形式进行跨域是不能直接的获取到返回的数据的，需要在data.php中返回一段js脚本，然后再获取此脚本，例如data.php返回的是`echo 'var data =123';`,在页面中可以使用如下的形式进行接收

```javascript
<script src="http://tom.com/data.php"></script>
<script>
	console.log(data);    
</script>
```
1.必须保证加载的顺序
2.不方便通过程序传递参数

- 动态创建script标签，通过标签src属性发送请求(JSONP的本质)

动态创建script标签发出去的请求是异步的请求

```javascript
var script = document.createElement('script');
var param = 'flag=1'
script.src = 'http://tom.com/data.php'+param;
var head = document.getElementByTagName('head')[0];
head.appendChild(sript);
//定义一个方法(服务端返回一个函数调用).这个函数是由服务器响应的内容调用（响应的内容就是js代码 ----foo(123)  )
function foo(data){
    console.log(data);
}
//console.log(data);
```

服务端的代码是：

```php
<?php
    echo 'foo(123)'
 ?>
```

> 服务端返回一个函数调用，这个是JsonP原理的关键

现在调用方法的名字不灵活，处理的方式为

```javascript
var script = document.createElement('script');
var param = 'flag=1'
// 使用callback传递回调函数的名字
script.src = 'http://tom.com/data.php?callback=abc'+param;
var head = document.getElementByTagName('head')[0];
head.appendChild(sript);
//定义一个方法(服务端返回一个函数调用).这个函数是由服务器响应的内容调用（响应的内容就是js代码 ----abc(123)  )
function abc(data){
    console.log(data);
}
//console.log(data);
```

> JSONP本质：动态的创建script标签，然后通过它的src属性发送跨域请求，然后服务器端响应的数据格式为【函数调用foo(参数)】，所以在发送请求之前需要先声明一个函数，并且这个函数的名字与参数中传递的名字需要一致。这里声明的函数是由服务器响应的内容（时间就是一段js的代码）来调用的

### 5.2 JSONP使用

```javascript
$(function(){
            $.ajax({
                type: "get",
                url: "http://tom.com/jsonp.php",
                dataType: "jsonp", //JSONP的时候是默认加上和调用函数的名称
              jsonp:'cb', //自定义参数名字（callback=abc 这里的名字指的是等号前面的键）
                // 后端根据这个键获取方法名，jQuery默认的是callback
               jsonpCallBack:'abc', //这个属性的作用是自定义回调函数的名字，等号后面的值
                success: function (response) {
                    console.log(response);
                }
            });
        });
```

服务端php的代码：

```php
<?php
    // 获取到函数的名字
    $cb = $_GET['callback'];
	$arr = array("username"=>"zhangsan","password"=>"123");
    echo $cb.'('.json_encode($arr).')';
 ?>
```

## 6. Ajax模板引擎

规范数据的解析过程，后期的维护非常的方便

模板引擎：模板+数据 —>静态页面

先看看原生的js数据解析：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="script/jquery-1.12.2.js"></script>
    <script src="script/template.js"></script>
    <script>
        window.onload = function () {
            function success(data) {
                // 这里就相当于之前的回调函数
                var title = data.title;
                var books = data.books;
                var tag = '<h1>' + title + '</h1>';
                for (var i = 0; i < books.length; i++) {
                    tag += '<div>' + books[i] + '</div>';
                }
                var con = document.getElementById("container");
                con.innerHTML = tag;
            }

            var data = {
                title: '图书信息',
                books: ['三国演义', '西游记', '水浒传', "红楼梦"]
            }
            success(data);
        }
    </script>

</head>

<body>
    <div id="container"></div>
</body>

</html>
```

实现的效果如下：

![mark](http://man.hhaxmm.cn/blog/20190421/h76wGIiMHDWl.png)

这样的方式进行数据解析，后期很难维护，因此使用模板引擎规范数据的解析过程，后期的维护非常的方便；常见的模板引擎有如下的几种：

![mark](http://man.hhaxmm.cn/blog/20190421/AMQw5XREkriV.png)

我主要学习的是art-template，它的教程地址为 [模板引擎](https://github.com/aui/art-template)

现在使用art-template模板引擎改造上面的代码：

**方式一：**

```html
<!--
 * @Description: art-template模板引擎
 * @Author: river
 * @LastEditors: huanghe
 * @Date: 2019-04-21 09:41:23
 * @LastEditTime: 2019-04-21 10:04:17
-->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="script/jquery-1.12.2.js"></script>
    <script src="script/template.js"></script>
    <!-- 使用模板引擎实现,id自己定义 -->
    <script id="tpl-user" type="text/html">
    
        <h1>{{title}}</h1>
        {{if books}}
            {{each books as value i}}
                <div>{{value}}</div>
            {{/each}}
        {{/if}}
    </script>
    <script>
        window.onload = function () {
            var data = {
                title: '图书信息',
                books: ['三国演义', '西游记', '水浒传', "红楼梦"]
            }
            // 1:渲染模板的id,2：数据
            var html = template('tpl-user',data);
            var con = document.getElementById("container");
            con.innerHTML = html;
        }
    </script>

</head>

<body>
    <div id="container"></div>
</body>

</html>
```

下面简单的说明一下art-template的语法

**方式二：**

使用渲染函数进行生成

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1>在javascript中存放模板</h1>
    <div id="container"></div>
    <script>
        var source = '<ul>'
        +    '{{each list as value i}}'
        +        '<li>索引 {{i+1}}:{{value}}</li>'
        +   '{{each}}'
        + '</ul>';
        // 根据模板生成渲染函数，compile方法返回值是一个函数
        var render = template.compile(source);
        // render作用就是用数据渲染静态标签
        var data = {
            list:['三国演义', '西游记', '水浒传', "红楼梦"];
        }
        var html = render(data);
        
        document.getElementById("container").innerHTML = html;
    </script>
</body>
</html>
```

> 更加倾向于使用方式一

### 5.1 art-template语法

art-template 支持标准语法与原始语法。标准语法可以让模板易读写，而原始语法拥有强大的逻辑表达能力。

标准语法支持基本模板语法以及基本 JavaScript 表达式；原始语法支持任意 JavaScript 语句，这和 EJS 一样。

#### **输出**

```html
{{value}}
//json样式的输出
{{data.key}} 
{{data['key']}}
{{a ? b : c}}
{{a || b}}
{{a + b}}
```

 **原始语法**

```
<%= value %>
<%= data.key %>
<%= data['key'] %>
<%= a ? b : c %>
<%= a || b %>
<%= a + b %>
```

模板一级特殊变量可以使用 `$data` 加下标的方式访问：

```
{{$data['user list']}}
```

#### **原文输出**

**标准语法**

```
{{@ value }}
```

**原始语法**

```
<%- value %>
```

> 原文输出语句不会对 `HTML` 内容进行转义处理，可能存在安全风险，请谨慎使用。

#### **条件**

**标准语法**

```
{{if value}} ... {{/if}}
{{if v1}} ... {{else if v2}} ... {{/if}}
```

**原始语法**

```
<% if (value) { %> ... <% } %>
<% if (v1) { %> ... <% } else if (v2) { %> ... <% } %>
```

#### 循环

**标准语法**

```
{{each target}}
    {{$index}} {{$value}}
{{/each}}
```

之前的版本是这样支持的：

```html
{{each books as value i}}
    <div>{{value}}</div>
{{/each}}
```

**原语法**

```
<% for(var i = 0; i < target.length; i++){ %>
    <%= i %> <%= target[i] %>
<% } %>
```

1. `target` 支持 `array` 与 `object` 的迭代，其默认值为 `$data`。
2. `$value` 与 `$index` 可以自定义：`{{each target val key}}`。

#### 变量

**标准语法**

```
{{set temp = data.sub.content}}
```

**原始语法**

```
<% var temp = data.sub.content; %>
```

#### 模板继承

**标准语法**

```
{{extend './layout.art'}}
{{block 'head'}} ... {{/block}}
```

**原始语法**

```
<% extend('./layout.art') %>
<% block('head', function(){ %> ... <% }) %>
```

模板继承允许你构建一个包含你站点共同元素的基本模板“骨架”。范例：

```html
<!--layout.art-->
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>{{block 'title'}}My Site{{/block}}</title>

    {{block 'head'}}
    <link rel="stylesheet" href="main.css">
    {{/block}}
</head>
<body>
    {{block 'content'}}{{/block}}
</body>
</html>
<!--index.art-->
{{extend './layout.art'}}

{{block 'title'}}{{title}}{{/block}}

{{block 'head'}}
    <link rel="stylesheet" href="custom.css">
{{/block}}

{{block 'content'}}
<p>This is just an awesome page.</p>
{{/block}}
```

渲染 index.art 后，将自动应用布局骨架。

#### 子模板

**标准语法**

```
{{include './header.art'}}
{{include './header.art' data}}
```

**原始语法**

```
<% include('./header.art') %>
<% include('./header.art', data) %>
```

1. `data` 数默认值为 `$data`；标准语法不支持声明 `object` 与 `array`，只支持引用变量，而原始语法不受限制。
2. art-template 内建 HTML 压缩器，请避免书写 HTML 非正常闭合的子模板，否则开启压缩后标签可能会被意外“优化。

例子：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

    <script id="test" type="text/html">
        <h1>{{title}}</h1>
        {{include 'list'}}
    </script>

    <script id='list' type="text/html">
        <ul>
            {{each list as value i}}
            <li>索引{{i+1}}:{{value}}</li>
            {{/each}}
        </ul>
    </script>

    <script>
        var data = {
            title: '图书信息',
            books: ['三国演义', '西游记', '水浒传', "红楼梦"]
        }
        // 1:渲染模板的id,数据
        var html = template('test', data);
        document.getElementById("container").innerHTML = html;
        
    </script>
</head>

<body>
</body>

</html>
```

