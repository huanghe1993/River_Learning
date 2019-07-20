## Vue全局过滤器和私有过滤器

### 私有过滤器

- 过滤器的名称
- 处理函数

过滤器调用的时候采用就近原则，如果私有的过滤器和全局过滤器的名称一致，先调用私有的，后全局的

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-05-15 17:30:32
 * @LastEditTime: 2019-05-16 11:41:07
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
    <link rel="stylesheet" href="./lib/bootstrap-3.3.7.css">
    <style type="text/css">

    </style>
</head>

<body>
    <div id="app">



        <div class="panel panel-primary">
            <div class="panel-heading">
                <h3 class="panel-title">添加平台</h3>
            </div>
            <div class="panel-body form-inline">

                <label>
                    Id:
                    <input type="text" class="form-control" v-model="id">
                </label>

                <label>
                    Name:
                    <input type="text" class="form-control" v-model="name">
                </label>

                <!-- 在vue中使用事件绑定机制为元素指定处理函数的时候，如果加了小括号可以传递参数 -->
                <input type="button" value="添加" class="btn btn-primary" @click='add'>

                <label>
                    搜索名称关键字:
                    <input type="text" class="form-control" v-model="keywords">
                </label>

            </div>
        </div>



        <table class="table table-hover table-bordered table-striped">
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Name</th>
                    <th>Ctime</th>
                    <th>Operation</th>
                </tr>
            </thead>
            <tbody>
                <!-- 指令可以得到任何的数据 -->
                <tr v-for="item in search(keywords)" :key="item.id">
                    <td>{{ item.id}}</td>
                    <td>{{ item.name}}</td>
                    <td>{{ item.ctime | dateFormate()}}</td>
                    <td>
                        <a href="" @click.prevent='del(item.id)'>删除</a>
                    </td>
                </tr>
            </tbody>
        </table>

    </div>

    <div id="app2">
        <h3>{{msg | dateFormate }}</h3>
    </div>
    <script>
        // 全局过滤器  时间格式化,pattern=""ES6语法的默认值，第一个参数是接受到的msg
        Vue.filter('dateFormate', function (dateStr, pattern = "") {
            // 根据给定的时间字符串，得到特定的时间
            var dt = new Date(dateStr);

            var y = dt.getFullYear();
            var m = dt.getMonth() + 1;
            var d = dt.getDay() + 1;

            if (pattern.toLowerCase() === 'yyyy-mm-dd') {
                return `${y}-${m}-${d}`;
            } else {
                var hh = dt.getHours();
                var mm = dt.getMinutes();
                var ss = dt.getSeconds();
                return `${y}-${m}-${d} ${hh}:${mm}:${ss}`;
            }
        });

        // 私有过滤器
        var vm2 = new Vue({
            el:'#app2',
            data: {
                msg: new Date(),
            },
            method: {},
            filters: { //私有过滤器
                dateFormate: function (dateStr, pattern = "") {
                    // 根据给定的时间字符串，得到特定的时间
                    var dt = new Date(dateStr);

                    var y = dt.getFullYear();
                    var m = dt.getMonth() + 1;
                    var d = dt.getDay() + 1;

                    if (pattern.toLowerCase() === 'yyyy-mm-dd') {
                        return `${y}-${m}-${d}`;
                    } else {
                        var hh = dt.getHours();
                        var mm = dt.getMinutes();
                        var ss = dt.getSeconds();
                        return `${y}-${m}-${d} ${hh}:${mm}:${ss} ----`;
                    }
                }

            }
        });

        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                id: '',
                name: '',
                keywords: '',
                list: [{
                        id: 1,
                        name: '奔驰',
                        ctime: new Date()
                    },
                    {
                        id: 2,
                        name: '宝马',
                        ctime: new Date()
                    },
                ]
            },
            methods: { //methods定义vue实例所有可用的方法
                add() {
                    // 添加的方法
                    this.list.push({
                        id: this.id,
                        name: this.name,
                        ctime: new Date()
                    });
                },
                del(id) {
                    // 根据id找索引
                    this.list.some((item, index) => {
                        if (item.id == id) {
                            // 从index处删除一个元素
                            this.list.splice(index, 1);
                            return true;
                        }
                    })
                },
                // 自定义search方法，把搜索的关键字通过传参的方式传给search方法
                search(keywords) {
                    var newList = [];
                    this.list.forEach(item => {
                        if (item.name.indexOf(keywords) != -1) {
                            newList.push(item);
                        }
                    });
                    return newList;
                }
            },
        });
    </script>
</body>

</html>
```

