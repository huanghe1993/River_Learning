## v-bind


**缩写：** :
**类型：** any (with argument) | Object (without argument)
**参数：** attrOrProp (optional)
**修饰符：**

- .prop - 被用于绑定 DOM 属性。(what’s the difference?)
- .camel - (2.1.0+) transform the kebab-case attribute name into camelCase. (supported since 2.1.0)
- .sync (2.3.0+) 语法糖，会扩展成一个更新父组件绑定值的 v-on 侦听器。

**用法：**
属性绑定机制

**代码示例1**：给按钮绑定属性

```html
 <div id="app">
        <!-- v-bind是vue中，提供用于绑定属性的指令 -->
        <input type="button" value="按钮" v-bind:title='myTitle'>
    </div>

    <script src="./lib/vue-2.4.0.js"></script>
    <script>
        var vm = new Vue({
            el: '#app',
            data: {
                myTitle: '自己定义的title'
            }
        });
    </script>
```

> 预览：<https://huanghe1993.github.io/chapter03/04v-bind1.html

**实际效果：**

<img src='http://man.hhaxmm.cn/blog/20190514/MjEOFj67e6yL.gif' align='left'>

------

**代码示例2**：缩略形式，v-bind中可以写合法的js表达式

```html
 <div id="app">
        <!-- v-bind缩略形式 -->
        <input type="button" value="按钮" :title="myTitle+'123'">
    </div>

    <script src="./lib/vue-2.4.0.js"></script>
    <script>
        var vm = new Vue({
            el: '#app',
            data: {
                myTitle: '自己定义的title'
            }
        });
    </script>
```

> 预览：<<https://huanghe1993.github.io/chapter03/04v-bind2.html
