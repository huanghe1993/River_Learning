Vue尝鲜

[TOC]

### 演示效果1

将 data 中的数据渲染到页面上。

> 预览 <https://huanghe1993.github.io/chapter01/02Vue-demo2.html>

<img src='http://man.hhaxmm.cn/blog/20190513/e4PC2JAFK38y.png' align='left' />

> 示例代码1：

```html
 <div id="app">
        {{message}}
        <hr />
        {{msg2}}
        <hr />
        {{msg}}
        <hr />
        {{arr}}
        <hr />
        {{json}}
    </div>
    <script src="./lib/vue-2.4.0.js" type="text/javascript" charset="utf-8"></script>
    <script type="text/javascript">
        var app = new Vue({
            el: '#app',
            data: {
                message: 'Hello Vue!',
                msg2:10,
                msg:true,
                arr:['apple','banner','orange'],
                json:{a:'张三',b:'李四',c:'王五'}
            }
        });
    </script>
```

### 演示效果2

实现数据双向绑定。

> 预览 https://huanghe1993.github.io/chapter01/02Vue-demo3.html

<img src='http://man.hhaxmm.cn/blog/20190513/VSg65MogAxJR.gif' align='left' />

> 示例代码2：

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-05-13 15:32:13
 * @LastEditTime: 2019-05-13 15:35:57
 * @LastEditors: huanghe
 -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #demo {
            width: 800px;
            margin: 200px auto;
        }

        input {
            width: 600px;
            height: 50px;
            border: 10px solid green;
            padding-left: 10px;
            font: 30px/50px "微软雅黑";
        }

        .msg {
            width: 600px;
            font: 30px/50px "微软雅黑";
            color: red;
        }
    </style>
    <script src="./lib/vue-2.4.0.js" type="text/javascript" charset="utf-8"></script>
    <script>
        window.onload = function () {
            new Vue({
                el: '#demo',
                data: {
                    msg: 'welcome vue.js',
                }
            })
        }
    </script>
</head>

<body>
    <div id="demo">
        <input v-model='msg'>
        <div class="msg">
            {{msg}}
        </div>
    </div>
</body>

</html>
```

将message绑定到文本框，当更改文本框的值时，`<p>{{ message }}</p>` 中的内容也会被更新。

上面用到的`v-model`是Vue.js常用的一个指令，那么指令是什么呢？

> **Vue.js的指令是以v-开头的，它们作用于HTML元素，指令提供了一些特殊的特性，将指令绑定在元素上时，指令会为绑定的目标元素添加一些特殊的行为，我们可以将指令看作特殊的HTML特性（attribute）。**

Vue.js具有良好的扩展性，我们也可以开发一些自定义的指令，后面的文章会介绍自定义指令。

### 演示效果3

渲染json数据。

> 预览：<https://huanghe1993.github.io/chapter01/02Vue-demo4.html>

<img src='http://man.hhaxmm.cn/blog/20190513/4JavRjzR0YVb.png' align='left' />

> 代码示例3

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-05-13 15:52:17
 * @LastEditTime: 2019-05-13 15:52:37
 * @LastEditors: huanghe
 -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <div id="app">
        <ol>
            <li v-for="list in msg">
                {{list.name}}
                {{list.age}}
                {{list.addr}}
            </li>
        </ol>

    </div>
    <script src="vue.js"></script>
    <script type="text/javascript">
        new Vue({
            el: '#app', //
            data: {
                msg: [{
                        name: '张三1',
                        age: '18',
                        addr: 'vue1'
                    },
                    {
                        name: '张三2',
                        age: '18',
                        addr: 'vue2'
                    },
                    {
                        name: '张三3',
                        age: '18',
                        addr: 'vue3'
                    },
                    {
                        name: '张三4',
                        age: '18',
                        addr: 'vue4'
                    },
                    {
                        name: '张三5',
                        age: '18',
                        addr: 'vue5'
                    },
                    {
                        name: '张三6',
                        age: '18',
                        addr: 'vue6'
                    }
                ]
            }
        });
    </script>
</body>

</html>
```

