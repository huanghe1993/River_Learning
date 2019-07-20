## [Vue中的动画](https://cn.vuejs.org/v2/guide/transitions.html)

为什么要有动画：动画能够提高用户的体验，帮助用户更好的理解页面中的功能；

![mark](http://man.hhaxmm.cn/blog/20190519/s2wsRoYASaPM.png)

动画分为：进入动画和离开动画

- 进入动画的起始状态：v-enter ；

- 进入动画的离开状态：v-enter-to;

- 进入动画的时间段：v-enter-active

> 使用过渡类名实现动画：

1. 使用transition元素，把需要被动画控制的元素，包裹起来，transition元素，是vue官方提供
2. 自定义两组样式，来控制transition内部元素实现动画

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue-2.4.0.js"></script>
    <!-- 2. 自定义两组样式，来控制 transition 内部的元素实现动画 -->
    <style>
        /* v-enter 【这是一个时间点】 是进入之前，元素的起始状态，此时还没有开始进入 */
        /* v-leave-to 【这是一个时间点】 是动画离开之后，离开的终止状态，此时，元素 动画已经结束了 */
        .v-enter,
        .v-leave-to {
            opacity: 0;
            transform: translateX(150px);
        }

        /* v-enter-active 【入场动画的时间段】 */
        /* v-leave-active 【离场动画的时间段】 */
        .v-enter-active,
        .v-leave-active {
            transition: all 0.8s ease;
        }
    </style>
</head>

<body>
    <div id="app">
        <input type="button" value="toggle" @click="flag=!flag">
        <!-- 需求： 点击按钮，让 h3 显示，再点击，让 h3 隐藏 -->
        <!-- 1. 使用 transition 元素，把 需要被动画控制的元素，包裹起来 -->
        <!-- transition 元素，是 Vue 官方提供的 -->
        <transition>
            <h3 v-if="flag">这是一个H3</h3>
        </transition>
    </div>

    <script>
        // 创建 Vue 实例，得到 ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                flag: false
            },
            methods: {}
        });
    </script>
</body>

</html>
```

> 在线演示：https://huanghe1993.github.io/chapter07/01vue的动画.html

实现的效果如下所示：

<img src = 'http://man.hhaxmm.cn/blog/20190519/XIenqV3kcmkL.gif' align ='left'>

> 自定义v-前缀（需求：H3淡入淡出，H6上下淡入淡出）

代码示例：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue-2.4.0.js"></script>
    <!-- 2. 自定义两组样式，来控制 transition 内部的元素实现动画 -->
    <style>
      /* v-enter 【这是一个时间点】 是进入之前，元素的起始状态，此时还没有开始进入 */
     /* v-leave-to 【这是一个时间点】 是动画离开之后，离开的终止状态，此时，元素 动画已经结束了 */
        .v-enter,
        .v-leave-to {
            opacity: 0;
            transform: translateX(150px);
        }

        /* v-enter-active 【入场动画的时间段】 */
        /* v-leave-active 【离场动画的时间段】 */
        .v-enter-active,
        .v-leave-active {
            transition: all 0.8s ease;
        }

        .my-enter,
        .my-leave-to {
            opacity: 0;
            transform: translateY(150px);
        }

        /* v-enter-active 【入场动画的时间段】 */
        /* v-leave-active 【离场动画的时间段】 */
        .my-enter-active,
        .my-leave-active {
            transition: all 0.8s ease;
        }
    </style>
</head>

<body>
    <div id="app">
        <input type="button" value="toggle" @click="flag=!flag">
        <!-- 需求： 点击按钮，让 h3 显示，再点击，让 h3 隐藏 -->
        <!-- 1. 使用 transition 元素，把 需要被动画控制的元素，包裹起来 -->
        <!-- transition 元素，是 Vue 官方提供的 -->
        <transition>
            <h3 v-if="flag">这是一个H3</h3>
        </transition>

        <hr>
        <input type="button" value="toggle2" @click="flag2=!flag2">
        <!-- 需求： 点击按钮，让 h3 显示，再点击，让 h3 隐藏 -->
        <!-- 1. 使用 transition 元素，把 需要被动画控制的元素，包裹起来 -->
        <!-- transition 元素，是 Vue 官方提供的 -->
        <transition name='my'>
            <h6 v-if="flag2">这是一个H6</h6>
        </transition>
        
    </div>

    <script>
        // 创建 Vue 实例，得到 ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                flag: false,
                flag2: false
            },
            methods: {}
        });
    </script>
</body>

</html>
```

> 在线演示：[https://huanghe1993.github.io/chapter07/02%E4%BF%AE%E6%94%B9v-%E5%89%8D%E7%BC%80.html](https://huanghe1993.github.io/chapter07/02修改v-前缀.html)

实现的效果如下所示：

<img src = 'http://man.hhaxmm.cn/blog/20190519/zblEgdXGjoMV.gif' align ='left'>

> 案例三：实现半场动画

可以在属性中声明 JavaScript 钩子

```html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

实现的具体代码：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script src="./lib/vue-2.4.0.js"></script>
  <style>
    .ball {
      width: 15px;
      height: 15px;
      border-radius: 50%;
      background-color: red;
    }
  </style>
</head>

<body>
  <div id="app">
    <input type="button" value="快到碗里来" @click="flag=!flag">
    <!-- 1. 使用 transition 元素把 小球包裹起来 -->
    <transition
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter">
      <div class="ball" v-show="flag"></div>
    </transition>
  </div>

  <script>

    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
      el: '#app',
      data: {
        flag: false
      },
      methods: {
        // 注意： 动画钩子函数的第一个参数：el，表示 要执行动画的那个DOM元素，是个原生的 JS DOM对象
        // 大家可以认为 ， el 是通过 document.getElementById('') 方式获取到的原生JS DOM对象
        beforeEnter(el){
          // beforeEnter 表示动画入场之前，此时，动画尚未开始，可以 在 beforeEnter 中，设置元素开始动画之前的起始样式
          // 设置小球开始动画之前的，起始位置
          el.style.transform = "translate(0, 0)"
        },
        enter(el, done){
          // 这句话，没有实际的作用，但是，如果不写，出不来动画效果；
          // 可以认为 el.offsetWidth 会强制动画刷新
          el.offsetWidth
          // enter 表示动画 开始之后的样式，这里，可以设置小球完成动画之后的，结束状态
          el.style.transform = "translate(150px, 450px)"
          el.style.transition = 'all 1s ease'

          // 这里的 done， 起始就是 afterEnter 这个函数，也就是说：done 是 afterEnter 函数的引用
          done()
        },
        afterEnter(el){
          // 动画完成之后，会调用 afterEnter
          // console.log('ok')
          this.flag = !this.flag
        }
      }
    });
  </script>
</body>

</html>
```

动画的实现的效果：

<img src = 'http://man.hhaxmm.cn/blog/20190519/aGFvqos3KIzu.gif' align ='left'>



