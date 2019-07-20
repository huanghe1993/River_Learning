## Vue路由的嵌套

实际生活中的应用界面，通常由多层嵌套的组件组合而成。同样地，URL 中各段动态路径也按某种结构对应嵌套的各层组件，例如：

```text
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

借助 `vue-router`，使用嵌套路由配置，就可以很简单地表达这种关系。

接着上节创建的 app：


```html
<div id="app">   
	<router-link to="/account">Account</router-link>
    <!-- 占坑使用的 -->
    <router-view></router-view>  
</div>
```

```js
var account = {
            template: '#tmpl'
		}
		// 路由
        var routerObj = new VueRouter({
            routes: [{
                    path: '/account',
                    component: account,  
               		},
            ]
        })
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {},
            methods: { //methods定义vue实例所有可用的方法 },
            // 绑定到app组件上
            router: routerObj,
        });
    </script>
```

这里的 `<router-view>` 是最顶层的出口，渲染最高级路由匹配到的组件。同样地，一个被渲染组件同样可以包含自己的嵌套 `<router-view>`。例如，在 `account` 组件的模板添加一个 `<router-view>`：

```HTML
 <template id="tmpl">
        <div>
            <h1>这是一个account组件</h1>
            <router-link to="/account/login">登录组件</router-link>
            <router-link to="/account/register">注册组件</router-link>
            <router-view></router-view>
        </div>
 </template>
```

要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

```js
// 路由
        var routerObj = new VueRouter({
            routes: [{
                    path: '/account',
                    component: account,
                    // 子路由,子路由前面不带“/”,否则以根路径开始请求
                    children:[
                        {path:'login',component:login},
                        {path:'register',component:register}
                    ]
                },
                // {path:'/account/login',component:login},
                // {path:'/account/register',component:register}

            ]
        })
```

**要注意，以 / 开头的嵌套路径会被当作根路径。 这让你充分的使用嵌套组件而无须设置嵌套的路径。**

你会发现，`children` 配置就是像 `routes` 配置一样的路由配置数组，所以呢，你可以嵌套多层路由。

> 总体的代码如下所示：

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-06-01 19:48:44
 * @LastEditTime: 2019-06-01 20:14:44
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

        <router-link to="/account">Account</router-link>
        <!-- 占坑使用的 -->
        <router-view></router-view>
        
    </div>

    <template id="tmpl">
        <div>
            <h1>这是一个account组件</h1>
            <router-link to="/account/login">登录组件</router-link>
            <router-link to="/account/register">注册组件</router-link>
            <router-view></router-view>
        </div>
    </template>

    <script>
        var account = {
            template: '#tmpl'
        }

        var login = {
            template: '<h3>login</h3>'
        }

        var register = {
            template: '<h3>register</h3>'
        }

        // 路由
        var routerObj = new VueRouter({
            routes: [{
                    path: '/account',
                    component: account,
                    // 子路由
                    children:[
                        {path:'login',component:login},
                        {path:'register',component:register}
                    ]
                },
                // {path:'/account/login',component:login},
                // {path:'/account/register',component:register}

            ]
        })
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {

            },
            methods: { //methods定义vue实例所有可用的方法

            },
            // 绑定到app组件上
            router: routerObj,
        });
    </script>
</body>

</html>
```

