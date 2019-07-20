# webpack与Vue

[TOC]



## 一、webpack中配置.vue组件页面的解析

1. 运行`cnpm i vue -S`将vue安装为运行依赖；
2. 由于在webpack中，推荐使用`.vue`组件模板文件定义组件，所以需要运行`cnpm i vue-loader vue-template-compiler -D`将解析转换vue的包安装为开发依赖；
3. 运行`cnpm i style-loader css-loader -D`将解析转换CSS的包安装为开发依赖，因为.vue文件中会写CSS样式；

4. 在`webpack.config.js`中，添加如下`module`规则：

```json
module: {

    rules: [

      { test: /\.css$/, use: ['style-loader', 'css-loader'] },

      { test: /\.vue$/, use: 'vue-loader' }

    ]

  }

```

5. 创建`.vue`结尾的组件页面：

```vue
    <template>

      <!-- 注意：在 .vue 的组件中，template 中必须有且只有唯一的根元素进行包裹，一般都用 div 当作唯一的根元素 -->

      <div>

        <h1>这是APP组件 - {{msg}}</h1>

        <h3>我是h3</h3>

      </div>

    </template>



    <script>

    // 注意：在 .vue 的组件中，通过 script 标签来定义组件的行为，需要使用 ES6 中提供的 export default 方式，导出一个vue实例对象

    export default {

      data() {

        return {

          msg: 'OK'

        }

      }

    }

    </script>



    <style scoped>

    h1 {

      color: red;

    }

    </style>

```

6. 创建`main.js`入口文件：

```js
    // 导入 Vue 组件

    import Vue from 'vue'



    // 导入 App组件

    import App from './components/App.vue'



    // 创建一个 Vue 实例，使用 render 函数，渲染指定的组件

    var vm = new Vue({

      el: '#app',

      render: c => c(App)

    });

```

## 二、在使用webpack构建的Vue项目中使用模板对象？

1. 在`webpack.config.js`中添加`resolve`属性：

```
resolve: {
    alias: {
      'vue$': 'vue/dist/vue.esm.js'
    }
  }
```



## 三、ES6中语法使用总结

1. 使用 `export default` 和 `export` 导出模块中的成员; 对应ES5中的 `module.exports` 和 `export`
2. 使用 `import ** from **` 和 `import '路径'` 还有 `import {a, b} from '模块标识'` 导入其他模块
3. 使用箭头函数：`(a, b)=> { return a-b; }`

## 四、export和export default

在 ES6中，也通过 规范的形式，规定了 ES6 中如何 导入 和 导出 模块

ES6中导入模块，使用   import 模块名称 from '模块标识符'    import '表示路径'

ES6 中，使用 export default 和 export 向外暴露成员：

```js
var info = {
  name: 'zs',
  age: 20
}
export default info
```

> 注意： export default 向外暴露的成员，可以使用任意的变量来接收
>
> 注意： 在一个模块中，export default 只允许向外暴露1次
>
> 注意： 在一个模块中，可以同时使用 export default 和 export 向外暴露成员

 ```js
export var title = '小星星'
export var content = '哈哈哈'
 ```

接收代码：

```js
import m222, { title as title123, content } from './test.js'
console.log(m222)
console.log(title123 + ' --- ' + content)
```

> 注意： 使用 export 向外暴露的成员，只能使用 { } 的形式来接收，这种形式，叫做 【按需导出】
>
> 注意： export 可以向外暴露多个成员， 同时，如果某些成员，我们在 import 的时候，不需要，则可以 不在 {}  中定义
>
> 注意： 使用 export 导出的成员，必须严格按照 导出时候的名称，来使用  {}  按需接收；
>
> 注意： 使用 export 导出的成员，如果 就想 换个 名称来接收，可以使用 as 来起别名；

## 五、在vue组件页面中，集成vue-router路由模块

[vue-router官网](https://router.vuejs.org/)

1. 导入路由模块：

```
import VueRouter from 'vue-router'
```

2. 安装路由模块：

```vue
Vue.use(VueRouter);
```

3. 导入需要展示的组件:

```
import login from './components/account/login.vue'

import register from './components/account/register.vue'

```

4. 创建路由对象:

```
var router = new VueRouter({

  routes: [

    { path: '/', redirect: '/login' },

    { path: '/login', component: login },

    { path: '/register', component: register }

  ]

});

```

5. 将路由对象，挂载到 Vue 实例上:

```
var vm = new Vue({

  el: '#app',

  // render: c => { return c(App) }

  render(c) {

    return c(App);

  },

  router // 将路由对象，挂载到 Vue 实例上

});

```

6. 改造App.vue组件，在 template 中，添加`router-link`和`router-view`：

```
    <router-link to="/login">登录</router-link>

    <router-link to="/register">注册</router-link>



    <router-view></router-view>

```



## 组件中的css作用域问题



## 抽离路由为单独的模块