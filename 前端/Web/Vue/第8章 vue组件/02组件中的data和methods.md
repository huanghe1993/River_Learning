## 组件中展示数据和响应事件

在组件中，`data`需要被定义为一个方法，例如：

1. 在组件中，`data`需要被定义为一个方法，还必须返回一个对象。例如：

```h
<body>
  <div id="app">
    <mycom1></mycom1>
  </div>

  <script>
    // 1. 组件可以有自己的 data 数据
    // 2. 组件的 data 和 实例的 data 有点不一样,实例中的 data 可以为一个对象,但是组件中的 data 必须是一个方法
    // 3. 组件中的 data 除了必须为一个方法之外,这个方法内部,还必须返回一个对象才行;
    // 4. 组件中 的data 数据,使用方式,和实例中的 data 使用方式完全一样!!!
    Vue.component('mycom1', {
      template: '<h1>这是全局组件 --- {{msg}}</h1>',
      data: function () {
      	// return的对象是独立的
        return {
          msg: '这是组件的中data定义的数据'
        }
      }
    })

    // 创建 Vue 实例，得到 ViewModel
    var vm = new Vue({
      el: '#app',
      data: {},
      methods: {}
    });
  </script>
</body>

```

2. 在子组件中，如果将模板字符串，定义到了script标签中，那么，要访问子组件身上的`data`属性中的值，需要使用`this`来访问；

