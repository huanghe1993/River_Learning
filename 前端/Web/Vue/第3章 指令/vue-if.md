## v-if指令

**类型：** any
**用法：** 根据表达式的值的真假条件渲染元素。在切换时元素及它的数据绑定 / 组件被销毁并重建。如果元素是 `<template>` ，将提出它的内容作为条件块。

> 示例

```html
<h1 v-if="ok">Yes</h1>
```

> 代码示例1

```html
<div id="app">
    <h1>Hello,Vue  js!!</h1>
    <h1 v-if='yes'>Yes!</h1>
    <h1 v-if='no'>No!</h1>
    <h1 v-if='age >=25'>Age:{{age}}</h1>
    <h1 v-if='name.indexOf("jack") >=0'>Name:{{name}}</h1>
</div>
<script type="text/javascript">
    var vm=new Vue({
        el:'#app',
            data:{
                yes:true,
                no:false,
                age:28,
                name:'abcdefg'
        }
    })
</script>
```

> 预览：

效果如图：

![img](https://box.kancloud.cn/6dd43656d6ec744d96fbaa34fd12d661_364x197.png)

- 这段代码使用了4个表达式：
- 数据的yes属性为true，所以"Yes!"会被输出；
- 数据的no属性为false，所以"No!"不会被输出；
- 运算式age >= 25返回true，所以"Age: 28"会被输出；
- 运算式name.indexOf('jack') >= 0返回false，所以"Name: keepfool"不会被输出。

> 注意：v-if指令是根据条件表达式的值来执行元素的插入或者删除行为。

这一点可以从渲染的HTML源代码看出来，面上只渲染了3个`<h1>`元素，v-if值为false的`<h1>`元素没有渲染到HTML。

![img](https://box.kancloud.cn/a63ab9efdec100d2c6ff0283e6404273_525x269.png)

为了再次验证这一点，可以在Chrome控制台更改age属性，使得表达式age >= 25的值为false，可以看到`<h1>`Age: 28`</h1>`元素被删除了。

![img](https://box.kancloud.cn/1ec7535040cb2f8eb5aeebf83a5bec6a_634x734.gif)

age是定义在选项对象的data属性中的，为什么Vue实例可以直接访问它呢？
这是因为每个Vue实例都会代理其选项对象里的data属性。

### v-if 与v-show

- v-if的特点：每次都会重新删除或是创建元素
- v-show的特点：每次不会重新进行DOM的删除和创建，只是切换了dispaly：none样式

```html
<div id="app">
        <input type="button" value="button" @click='toggle'>

        <h3 v-if="flag">这是v-if控制的块</h3>
        <h3 v-show="flag">这是v-show控制的块</h3>
    </div>
    <script>
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                flag:true
            },
            methods: { //methods定义vue实例所有可用的方法
                toggle(){
                    this.flag = !this.flag;
                }
            },
        });
    </script>
```



## `<template>` 中 v-if 条件组

> 代码示例2

```html
<div id="demo">
    	<template v-if="login">
		  <h1>Title</h1>
		  <p>Paragraph 1</p>
		  <p>Paragraph 2</p>
		</template>
    </div>
    <script>
        var app=new Vue({
            el:'#demo',
            data:{
                login:true 
            }
        });
    </script>
```

因为 v-if 是一个指令，需要将它添加到一个元素上。但是如果我们想切换多个元素呢？此时我们可以把一个`<template>`元素当做包装元素，并在上面使用 v-if。最终的渲染结果不会包含 `<template>` 元素。
**效果如图：**
![img](https://box.kancloud.cn/f9b70a91512e1bd49735f2e5e19137fd_421x485.png)