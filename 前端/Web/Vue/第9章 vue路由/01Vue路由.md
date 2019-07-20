## 路由

1. **后端路由：**对于普通的网站，所有的超链接都是URL地址，所有的URL地址都对应服务器上对应的资源；
2. **前端路由：**对于单页面应用程序来说，主要通过URL中的hash(#号)来实现不同页面之间的切换，同时，hash有一个特点：HTTP请求中不会包含hash相关的内容；所以，单页面程序中的页面跳转主要用hash实现；
3. 在单页面应用程序中，这种通过hash改变来切换页面的方式，称作前端路由（区别于后端路由）；

[URL中的hash（井号）](http://www.cnblogs.com/joyho/articles/4430148.html)

#### 安装

- 直接下载 / CDN

<https://unpkg.com/vue-router/dist/vue-router.js>

在 Vue 后面加载 `vue-router`，它会自动安装的：

```html
<script src="/path/to/vue.js"></script>
<script src="/path/to/vue-router.js"></script>
```

- NPM

```bash
npm install vue-router
```

如果在一个模块化工程中使用它，必须要通过 `Vue.use()` 明确地安装路由功能：

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

如果使用全局的 script 标签，则无须如此 (手动安装)

#### 基本的使用

1. 导入vue-router模块的包

2. 创建路由对象，当导入包之后，在window全局对象中，就有一个路由的构造函数，叫VueRouter。在new路由对象的时候可以为构造函数传递，一个配置对象

   2.1. 路由匹配规则,每个路由规则都是一个对象，这个对象身上有两个必须的属性

   2.2 属性1：path,表示监听哪个路由链接地址

   2.3 属性2：component,表示，如果路由是前面匹配到的path,则展示component属性对应的那个组件
3. 将路由规则对象注册到VM实例上，用来监听URl地址的变化，然后展示对应的组件


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title></title>
    <script src="./lib/vue-2.4.0.js"></script>
    <!-- 1、vue-router模块 -->
    <script src="./lib/vue-router-3.0.1.js"></script>
</head>

<body>
    <div id="app">

        <!-- 登录 -->
        <a href="#/login">登录</a>
        <a href="#/register">注册</a>


        <!-- 路由的展示，vue-router提供的元素，专门用来当作占位符的，将来路由规则匹配到的组件，就会展示到这个route-view中去 -->
        <!-- 所以可以认为route-view是占位符 -->
        <router-view></router-view>
    </div>

    <script>
        // 创建登录模板对象
        var login = {
            template:'<h1>登录组件</h1>'
        }

        var register = {
            template:'<h1>注册组件</h1>'
        }

        
        // 2、创建路由对象，当导入包之后，在window全局对象中，就有一个路由的构造函数，叫VueRouter
        // 在new路由对象的时候可以为构造函数传递，一个配置对象，
        var routerObj = new VueRouter({
            //路由匹配规则
            routes:[
                // 每个路由规则都是一个对象，这个对象身上有两个必须的属性
                // 属性1：path,表示监听哪个路由链接地址
                // 属性2：component,表示，如果路由是前面匹配到的path,则展示component属性对应的那个组件，
                // 注意，component的属性值，必须是组件的模板对象，不能是组件模板名称
                {path:'/login',component: login},
                {path:'/register',component:register}
            ]  
            
        })




        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                
            },
            methods: { //methods定义vue实例所有可用的方法
                
            },
            // 3、将路由规则对象注册到VM实例上，用来监听URl地址的变化，然后展示对应的组件
            router:routerObj,
        });
    </script>
</body>
</html>
```

#### route-link的使用

他的使用用来替代链接中的`#`

```html
<a href="#/login">登录</a>
<a href="#/register">注册</a>
```

替代的方法

```html
<router-link to='/login'>登录</router-link>
<router-link to='/register'>注册</router-link>
```

#### 页面重定向redirect

```js
 // 默认的展示页面,页面重定向
 {path:'/',redirect:"/login"},
```

#### 设置选中路由高亮显示

第一种：修改别人原生提供的类进行修改样式

```html
<style type="text/css">
        .router-link-active{
            color: red;
            font-weight: 800;
            font-style: italic;
            font-size: 80px;
        }
</style>
```

第二种：修改类名为自己的类名

```js
 var routerObj = new VueRouter({
            //路由匹配规则
            routes:[
                // 每个路由规则都是一个对象，这个对象身上有两个必须的属性
                // 属性1：path,表示监听哪个路由链接地址
                // 属性2：component,表示，如果路由是前面匹配到的path,则展示component属性对应的那个组件，
                // 注意，component的属性值，必须是组件的模板对象，不能是组件模板名称、
                {path:'/login',component: login},
                {path:'/register',component:register},
                // 默认的展示页面,页面重定向
                {path:'/',redirect:"/login"},
            ],
            // 修改
            linkActiveClass:'myactive'            
        })
```

#### 使用query方式传递参数

- this.$route.query.id

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-05-29 21:21:52
 * @LastEditTime: 2019-05-29 21:49:17
 * @LastEditors: huanghe
 -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title></title>
    <script src="./lib/vue-2.4.0.js"></script>
    <script src="./lib/vue-router-3.0.1.js"></script>
    <style type="text/css">

    </style>
</head>

<body>
    <div id="app">
        <!-- 如果在路由中，使用查询字符串，给路由传递参数，则不需要修改路由规则的path属性 -->
        <router-link to='/login?id=10' tag="span">登录</router-link>
        <router-link to='/register' tag="span">注册</router-link>

        <router-view></router-view>
    </div>
    <script>
        var login = {
            template: '<h1>登录</h1>',
            // 组件的钩子函数
            created() {
                // 获取地址栏中传递过来的参数
                console.log(this.$route.query.id);
            },
        }

        var register = {
            template: '<h1>注册组件</h1>'
        }

        var routerObj = new VueRouter({
            routes: [
                {path: '/login',component: login},
                {path: '/register',component: register},
                // 默认的展示页面,页面重定向
                {path: '/',redirect: "/login"},
            ]

        })

        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {

            },
            methods: { //methods定义vue实例所有可用的方法

            },
            router:routerObj,  
        });
    </script>
</body>

</html>
```

- 使用parm传递参数

```js
 routes: [
                {path: '/login/:id',component: login},
                {path: '/register',component: register},
                // 默认的展示页面,页面重定向
                {path: '/',redirect: "/login"},
            ]
```

使用方式，传递参数

```html
<router-link to='/login/12' tag="span">登录</router-link>
```

获取到参数的值

```js
 var login = {
            template: '<h1>登录---{{$route.params.id}}</h1>',
            // 组件的钩子函数
            created() {
                // 获取地址栏中传递过来的参数
                console.log(this.$route.params.id);
            },
        }
```



