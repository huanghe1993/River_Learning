## 过滤器

概念：Vue.js 允许你自定义过滤器，**可被用作一些常见的文本格式化**。过滤器可以用在两个地方：**mustache 插值和 v-bind 表达式**。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符指示；

```js
{{ name | 过滤器的名称 }}
```

### 定义过滤器

```html
 Vue.filter('过滤器的名称',function(data){})
```

> 示例1：过滤器的基本使用

```html
<div id="app">
        <p>{{msg | msgFormat}}</p>
    </div>
    <script>

        // 定义一个全局的过滤器
        Vue.filter('msgFormat',function(msg){
            //replace第一个参数可以是正则，全局匹配/案例/g
            return msg.replace("案例","例子")
        })


        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                msg:'过滤器的基本的使用案例！',
            },
            methods: { //methods定义vue实例所有可用的方法
                
            },
        });
    </script>
```

> 示例2：过滤器传递参数，并且调用多个过滤器

```html
<div id="app">
        <!--过滤器传递参数，并且调用多个过滤器  -->
        <p>{{msg | msgFormat('123','例子') | test }}</p>
    </div>
    <script>
        
        Vue.filter('msgFormat', function (msg, arg1, arg2) {
            return msg.replace(/案例/g, arg1 + arg2);
        })

        Vue.filter('test', function (msg) {
            return msg + "======";
        })

        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                msg: '过滤器的基本的使用案例！',
            },
            methods: { //methods定义vue实例所有可用的方法

            },
        });
    </script>
```

