## vue生命周期

- 什么是生命周期：从Vue实例创建、运行、到销毁期间，总是伴随着各种各样的事件，这些事件，统称为生命周期！

- [生命周期钩子](https://cn.vuejs.org/v2/api/#选项-生命周期钩子)：就是生命周期事件的别名而已；
+ 生命周期钩子 = 生命周期函数 = 生命周期事件

+ 主要的生命周期函数分类：

  - 创建期间的生命周期函数：

    - beforeCreate：实例刚在内存中被创建出来，此时，还没有初始化好 data 和 methods 属性

    - created：实例已经在内存中创建OK，此时 data 和 methods 已经创建OK，此时还没有开始编译模板

    - beforeMount：此时已经完成了模板的编译，但是还没有挂载到页面中

    - mounted：此时，已经将编译好的模板，挂载到了页面指定的容器中显示

  - 运行期间的生命周期函数：
  
    - beforeUpdate：状态更新之前执行此函数， 此时 data 中的状态值是最新的，但是界面上显示的 数据还是旧的，因为此时还没有开始重新渲染DOM节点
  
    - updated：实例更新完毕之后调用此函数，此时 data 中的状态值 和 界面上显示的数据，都已经完成了更新，界面已经被重新渲染好了！
  
  - 销毁期间的生命周期函数：
  
    - beforeDestroy：实例销毁之前调用。在这一步，实例仍然完全可用。
  - destroyed：Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。

![mark](http://man.hhaxmm.cn/blog/20190517/4N2dNtW2N2NI.png)

代码演示：

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-05-17 15:57:06
 * @LastEditTime: 2019-05-17 15:57:31
 * @LastEditors: huanghe
 -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue-2.4.0.js"></script>
</head>

<body>
    <div id="app">
        <input type="button" value="修改msg" @click="msg='No'">
        <h3 id="h3">{{ msg }}</h3>
    </div>

    <script>
        // 创建 Vue 实例，得到 ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                msg: 'ok'
            },
            methods: {
                show() {
                    console.log('执行了show方法')
                }
            },
            beforeCreate() { // 这是我们遇到的第一个生命周期函数，表示实例完全被创建出来之前，会执行它
                // console.log(this.msg)
                // this.show()
                // 注意： 在 beforeCreate 生命周期函数执行的时候，data 和 methods 中的 数据都还没有没初始化
            },
            created() { // 这是遇到的第二个生命周期函数
                // console.log(this.msg)
                // this.show()
                //  在 created 中，data 和 methods 都已经被初始化好了！
                // 如果要调用 methods 中的方法，或者操作 data 中的数据，最早，只能在 created 中操作
            },
            beforeMount() { // 这是遇到的第3个生命周期函数，表示 模板已经在内存中编辑完成了，但是尚未把 模板渲染到 页面中
                // console.log(document.getElementById('h3').innerText)
                // 在 beforeMount 执行的时候，页面中的元素，还没有被真正替换过来，只是之前写的一些模板字符串
            },
            mounted() { // 这是遇到的第4个生命周期函数，表示，内存中的模板，已经真实的挂载到了页面中，用户已经可以看到渲染好的页面了
                // console.log(document.getElementById('h3').innerText)
                // 注意： mounted 是 实例创建期间的最后一个生命周期函数，当执行完 mounted 就表示，实例已经被完全创建好了，此时，如果没有其它操作的话，这个实例，就静静的 躺在我们的内存中，一动不动
            },


            // 接下来的是运行中的两个事件
            beforeUpdate() { // 这时候，表示 我们的界面还没有被更新【数据被更新了吗？  数据肯定被更新了】
                /* console.log('界面上元素的内容：' + document.getElementById('h3').innerText)
                console.log('data 中的 msg 数据是：' + this.msg) */
                // 得出结论： 当执行 beforeUpdate 的时候，页面中的显示的数据，还是旧的，此时 data 数据是最新的，页面尚未和 最新的数据保持同步
            },
            updated() {
                console.log('界面上元素的内容：' + document.getElementById('h3').innerText)
                console.log('data 中的 msg 数据是：' + this.msg)
                // updated 事件执行的时候，页面和 data 数据已经保持同步了，都是最新的
            }
        });
    </script>
</body>

</html>
```

详细的图解：





