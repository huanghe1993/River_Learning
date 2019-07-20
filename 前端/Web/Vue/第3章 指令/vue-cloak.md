## v-cloak

**类型：** string
**用法：** 更新元素的 innerHTML 。

> 这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 `[v-cloak] { display: none }` 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕，能解决插值表达式的闪烁问题

HTML 绑定 Vue实例，在页面加载时会闪烁

> 代码示例1

```html
    <div id="app">
        {{msg}}
    </div>
    <script src="./lib/vue-2.4.0.js"></script>
    <script>
        var vm =new Vue({
            el:'#app',
            data:{
                msg:'123'
            }
        });
    </script>
```

> 预览：<<https://huanghe1993.github.io/chapter03/03v-cloak1.html>

<img src='http://man.hhaxmm.cn/blog/20190514/gQ3TM7IQy42F.gif' align='left'>

HTML 绑定 Vue实例，在页面加载时会闪烁然后才会出现 `123` 字样，

------

v-cloak 可以解决这一问题，在 css 中加上

```css
[v-cloak] {
  display: none;
}
```

在 html 中的加载点加上 v-cloak，就可以解决这一问题

```html
<div id="app" v-cloak>
    {{msg}}
</div>
```

> 代码示例2

```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        [v-cloak] {
            display: none;
        }
    </style>
</head>
<body>
    <div id="app" v-cloak>
        {{msg}}
    </div>
    <script src="./lib/vue-2.4.0.js"></script>
    <script>
        var vm =new Vue({
            el:'#app',
            data:{
                msg:'123'
            }
        });
    </script>
</body>
```

> 预览：<<https://huanghe1993.github.io/chapter03/03v-cloak2.html>