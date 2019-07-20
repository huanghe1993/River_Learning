## v-for指令

**类型：** Array | Object | number | string
**用法：** 基于源数据多次渲染元素或模板块。此指令之值，必须使用特定语法 alias in expression ，为当前遍历的元素提供别名：

> 示例：

```html
<div v-for="item in items">
  {{ item.text }}
</div>
```

> 代码示例1：

```html
		<div id="demo">
			<ol>
				<li v-for='value in arr'>
					{{value}} 
				</li>
			</ol>
		</div>
		<script type="text/javascript">
			new Vue({
				el: '#demo',
				data: {
				    arr:['apple','banana','orange','pear'],
				    json:{a:'张三',b:'李四',c:'王五'}
				}
			})
		</script>
```

> 预览：<>

效果如图：
![img](https://box.kancloud.cn/a279ffb75fcb134a898c8bbc1b91eb1a_291x261.png)

------

> 代码示例2：

```html
		<div id="demo">
			<ol>
				<li v-for='value in arr'>
					{{value}}
				</li>
			</ol>
			<hr />
			<ol>
				<li v-for='(value,index) in arr'>
					{{value}} {{index}}
				</li>
			</ol>
			<hr />
			<ol>
                <!--值和索引-->
				<li v-for='(item,index) in json'>
					{{item}}  {{index}} 
				</li>
			</ol>
			<hr />
			<ol>
                <!--val是值，key是键-->
				<li v-for='(val,key) in json'>
					{{val}}  {{key}} 
				</li>
			</ol>
			<hr />
			<ol>
                <!--val是值，key是键，index是索引-->
				<li v-for='(val,key,index) in json'>
					{{val}} {{key}} {{index}}
				</li>
			</ol>
		</div>
		<script type="text/javascript">
			new Vue({
				el: '#demo',
				data: {
				    arr:['apple','banana','orange','pear'],
				    json:{a:'张三',b:'李四',c:'王五'}
				}
			})
		</script>
```

> 预览：<>
> 效果如图：
> ![img](https://box.kancloud.cn/09c3dce9653c5c6fdc1515a430e99d42_240x597.png)

------

下面我们通过v-for来实现一个用户信息表

![img](https://box.kancloud.cn/fe54f48330d40968cd2e34617522bd6d_639x257.png)

> 预览：<>

> 附完整代码

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style>
			h1{
				text-align: center;
			}
			table,
			td,
			th {
				border-collapse: collapse;
				border-spacing: 0
			}
			table {
				width: 600px;
				margin: 0 auto;
			}
			td,
			th {
				border: 1px solid #bcbcbc;
				padding: 5px 10px;
			}
			
			th {
				background: #42b983;
				font-size: 1.2rem;
				font-weight: 400;
				color: #fff;
				cursor: pointer;
			}
			
			tr:nth-of-type(odd) {
				background: #fff;
			}
			
			tr:nth-of-type(even) {
				background: #eee;
			}
		</style>
		
	</head>
	<body>
		<div id="app">
			<h1>用户信息表</h1>
			<table>
				<thead>
					<tr>
						<th>Name</th>
						<th>Age</th>
						<th>Sex</th>
						<th>Addr</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="person in people">
						<td>{{ person.name  }}</td>
						<td>{{ person.age  }}</td>
						<td>{{ person.sex  }}</td>
						<td>{{ person.addr  }}</td>
					</tr>
				</tbody>
			</table>
		</div>
	</body>
	<script src="vue.js"></script>
	<script>
		var vm = new Vue({
			el: '#app',
			data: {
				people: [{
					name: 'Jack',
					age: 30,
					sex: 'Male',
					addr:'重庆'
				}, {
					name: 'Bob',
					age: 18,
					sex: 'Male',
					addr:'纽约'
					
				}, {
					name: 'Nini',
					age: 22,
					sex: 'Female',
					addr:'北京'
					
				}, {
					name: 'Mary',
					age: 16,
					sex: 'Male',
					addr:'上海'
				}]
			}
		})
	</script>
</html>
```

注意v-for中key的使用细节

```html
<!--
 * @Description: 
 * @Author: river
 * @Date: 2019-05-15 17:15:03
 * @LastEditTime: 2019-05-15 17:15:06
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

        <div>
            <label>Id:
                <input type="text" v-model="id">
            </label>

            <label>Name:
                <input type="text" v-model="name">
            </label>

            <input type="button" value="添加" @click="add">
        </div>

        <!-- 注意： v-for 循环的时候，key 属性只能使用 number获取string -->
        <!-- 注意： key 在使用的时候，必须使用 v-bind 属性绑定的形式，指定 key 的值 -->
        <!-- 在组件中，使用v-for循环的时候，或者在一些特殊情况中，如果 v-for 有问题，必须 在使用 v-for 的同时，指定 唯一的 字符串/数字 类型 :key 值 -->
        <p v-for="item in list" :key="item.id">
            <input type="checkbox">{{item.id}} --- {{item.name}}
        </p>
    </div>

    <script>
        // 创建 Vue 实例，得到 ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                id: '',
                name: '',
                list: [{
                        id: 1,
                        name: '李斯'
                    },
                    {
                        id: 2,
                        name: '嬴政'
                    },
                    {
                        id: 3,
                        name: '赵高'
                    },
                    {
                        id: 4,
                        name: '韩非'
                    },
                    {
                        id: 5,
                        name: '荀子'
                    }
                ]
            },
            methods: {
                add() { // 添加方法
                    this.list.unshift({
                        id: this.id,
                        name: this.name
                    })
                }
            }
        });
    </script>
</body>

</html>
```

