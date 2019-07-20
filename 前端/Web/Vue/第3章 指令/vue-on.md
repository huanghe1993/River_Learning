## v-on指令

**缩写：** @
**类型：** Function | Inline Statement
**参数：** event (required)
**事件修饰符：**

- .stop - 调用 event.stopPropagation()。阻止冒泡
- .prevent - 调用 event.preventDefault()。阻止默认事件
- .capture - 添加事件侦听器时使用 capture 模式。使用捕获机制
- .self - 只当事件是从侦听器绑定的元素本身触发时才触发回调。
- .{keyCode | keyAlias} - 只当事件是从特定键触发时才触发回调。
- .native - 监听组件根元素的原生事件。
- .once - 只触发一次回调。
- .left - (2.2.0) 只当点击鼠标左键时触发。
- .right - (2.2.0) 只当点击鼠标右键时触发。
- .middle - (2.2.0) 只当点击鼠标中键时触发。
- .passive - (2.3.0) 以 { passive: true } 模式添加侦听器

**用法：**
绑定事件监听器。事件类型由参数指定。表达式可以是一个方法的名字或一个内联语句，如果没有修饰符也可以省略。

```html
<!-- 方法处理器 -->
<!------指令：参数 --->
<button v-on:click="doThis"></button>
<!-- 内联语句 -->
<button v-on:click="doThat('hello', $event)"></button>
<!-- 缩写 -->
<button @click="doThis"></button>
<!-- 停止冒泡 -->
<button @click.stop="doThis"></button>
<!-- 阻止默认行为 -->
<button @click.prevent="doThis"></button>
<!-- 阻止默认行为，没有表达式 -->
<form @submit.prevent></form>
<!--  串联修饰符 -->
<button @click.stop.prevent="doThis"></button>
<!-- 键修饰符，键别名 -->
<input @keyup.enter="onEnter">
<!-- 键修饰符，键代码 -->
<input @keyup.13="onEnter">
<!-- 点击回调只会触发一次 -->
<button v-on:click.once="doThis"></button>
```

在子组件上监听自定义事件（当子组件触发 “my-event” 时将调用事件处理器）：

```html
<my-component @my-event="handleThis"></my-component>
<!-- 内联语句 -->
<my-component @my-event="handleThis(123, $event)"></my-component>
<!-- 组件中的原生事件 -->
<my-component @click.native="onClick"></my-component>
```

> 代码示例1：弹出输出内容

```html
<div id="app">
        <input type="button" value="按钮" v-on:click='show'>
    </div>
    <script src="./lib/vue-2.4.0.js"></script>
    <script>
        var vm = new Vue({
            el: '#app',
            data: {
                myTitle: '自己定义的title'
            },
            methods: { //methods定义vue实例所有可用的方法
                show:function(){
                    alert('hello');
                }
            },
        });
    </script>
```

> 代码示例2：跑马灯

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>跑马灯的效果</title>
    <script src="./lib/vue-2.4.0.js"></script>
</head>

<body>
    <div id="app">
        <input type="button" value="浪起来" @click='lang'>
        <input type="button" value="低调" @click='stop'>
        <h4>{{msg}}</h4>
    </div>

    <script>
        var vm = new Vue({
            el: '#app',
            data: {
                msg: '畏缩发育别浪~~~!',
                intervalId:null //在data中定义定时器id
            },
            methods: {
                lang: function () {
                    //setInterval和setTimeout的回调函数中this的指向都是window,所以
                    //这里先对当前的对象保留一份
                    var _this = this;
                    if (this.intervalId != null) return;
                    //使用箭头函数，内部的this永远指向外部的this,所以此时可以使用this
                    this.intervalId = setInterval(() => {
                        // 拿到msg数据,使用this，代表当前实例
                        var start = _this.msg.substring(0, 1);
                        var end = _this.msg.substring(1);
                        // 重新赋值,vm实例，会监听自己身上data中所有数据的改变
                        //只要数据发生变化，就会自动把最新的数据从data中同步到页面
                        _this.msg = end + start;
                    }, 500);

                },
                stop(){
                    clearInterval(this.intervalId);
                    // 清除定时器之后重新置intervalId为null
                    this.intervalId =null;

                }
            },
        });
    </script>
</body>

</html>
```

**实现效果：**

<img src='http://man.hhaxmm.cn/blog/20190514/1IWa4ApXxgnL.gif' align='left'>

> 示例代码：事件修饰符

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-05-15 10:27:14
 * @LastEditTime: 2019-05-15 15:28:00
 * @LastEditors: huanghe
 -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>事件修饰符</title>
    <script src="./lib/vue-2.4.0.js"></script>
    <style type="text/css">
        .inner {
            height: 150px;
            background-color: darkcyan;
        }
    </style>
</head>

<body>
    <div id="app">
        <div class="inner" @click="div1Handler">
            <!--.stop阻止冒泡 -->
            <input type="button" value="戳" @click.stop  = "btnHandler">
        </div>
        
        <!-- prevent阻止默认的行为 -->
        <a href="http://www.baidu.com" @click.prevent="linkClick">百度</a>
    </div>
    <script>
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                
            },
            methods: { //methods定义vue实例所有可用的方法
                div1Handler(){
                    console.log("触发inner div的点击事件");
                },
                btnHandler(){
                    console.log("触发btn的点击事件");
                },
                linkClick(){
                    console.log("跳转到了百度");
                }
            },
        });
    </script>
</body>
</html>
```

