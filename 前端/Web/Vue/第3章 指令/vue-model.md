## v-model指令

**类型：** 随表单控件类型不同而不同。
**修饰符：**

- .lazy - 取代 input 监听 change 事件
- .number - 输入字符串转为数字
- .trim - 输入首尾空格过滤

**用法：** v-model指令用来在input、select、text、checkbox、radio等表单控件或者组件上创建双向绑定。
你可以用 v-model 指令在表单控件元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。尽管有些神奇，但 v-model 本质上不过是语法糖，它负责监听用户的输入事件以更新数据，并特别处理一些极端的例子。

> v-bind只能实现数据的单向绑定（M--->V），v-model可以实现数据的双向绑定

> 示例代码1

```html
<div id="app">
        <h4>{{msg}}</h4>

        <!--v-model可以实现表单元素和Model中数据的双向绑定，只能用在表单元素  -->
        <input type="text" name="" id="" v-model="msg">
    </div>
    <script>
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                msg:"好好的学习vue",
            },
            methods: { //methods定义vue实例所有可用的方法
                
            },
        });
    </script>
```

> 效果展示：

![mark](http://man.hhaxmm.cn/blog/20190515/UJCX5qsFEiC8.gif)

> 示例2：计算器

```html
<div id="app">
        <input type="text" name="" id="" v-model="n1">
        <select name="" id="" v-model="opt">
            <option value="+">+</option>
            <option value="-">-</option>
            <option value="*">*</option>
            <option value="/">/</option>
        </select>
        <input type="text" name="" id="" v-model="n2">

        <input type="button" value="=" @click="calc">

        <input type="text" name="" id="" v-model="result">
    </div>
    <script>
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                n1: 0,
                n2: 0,
                result: 0,
                opt: '+'
            },
            methods: { //methods定义vue实例所有可用的方法
                calc() {
                    var codeStr = 'parseInt(this.n1)'+this.opt+'this.opt+parseInt(this.n2)'
                    // 字符串转化为数字计算
                    this.result = eval(codeStr)
                }
            },
        });
    </script>
```

