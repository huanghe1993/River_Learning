## MVVM是什么？

MVVM 是Model-View-ViewModel 的缩写，它是一种基于前端开发的架构模式，其核心是提供对View 和 ViewModel 的双向数据绑定，这使得ViewModel 的状态改变可以自动传递给 View，即所谓的数据双向绑定。

![img](https://box.kancloud.cn/eb5c6e9f9bb7107c374dd892f6d7d38f_600x287.png)

Vue.js 是一个提供了 MVVM 风格的双向数据绑定的 Javascript 库，专注于View 层。它的核心是 MVVM 中的 VM，也就是 ViewModel。 ViewModel负责连接 View 和 Model，保证视图和数据的一致性，这种轻量级的架构让前端开发更加高效、便捷。

### Vue 双向绑定原理

Vue.js 是采用[ Object.defineProperty ](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)的 getter 和 setter，并结合观察者模式来实现数据绑定的。当把一个普通 Javascript 对象传给 Vue 实例来作为它的 data 选项时，Vue 将遍历它的属性，用 Object.defineProperty 将它们转为 getter/setter。用户看不到 getter/setter，但是在内部它们让 Vue 追踪依赖，在属性被访问和修改时通知变化。

![img](https://box.kancloud.cn/04772e55186c0805646fe1bd1696d5d3_647x334.png)

> 预览：<https://huanghe1993.github.io/chapter02/js-mvvm.html>

> 代码示例如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>River-Vue双向数据绑定实现原理</title>
</head>
<body>
    <div id="app">
        <input type="text" v-model="text">
        {{ text }}
    </div>

    <script>
        function observe (obj, vm) {
            Object.keys(obj).forEach(function (key) {
                defineReactive(vm, key, obj[key]);
            });
        }
        function defineReactive (obj, key, val) {
            var dep = new Dep();
            Object.defineProperty(obj, key, {
                get: function () {
                    // 添加订阅者watcher到主题对象Dep
                    if (Dep.target) dep.addSub(Dep.target);
                    return val
                },
                set: function (newVal) {
                    if (newVal === val) return
          　　　　　　val = newVal;
                    // 作为发布者发出通知
                    dep.notify();
                }
            });
        }
        function nodeToFragment (node, vm) {
            var flag = document.createDocumentFragment();
            var child;
            while (child = node.firstChild) {
                compile(child, vm);
                flag.append(child); // 将子节点劫持到文档片段中
            }
            return flag;
        }
        function compile (node, vm) {
            var reg = /\{\{(.*)\}\}/;
            // 节点类型为元素
            if (node.nodeType === 1) {
                var attr = node.attributes;
                // 解析属性
                for (var i = 0; i < attr.length; i++) {
                    if (attr[i].nodeName == 'v-model') {
                        var name = attr[i].nodeValue; // 获取v-model绑定的属性名
                        node.addEventListener('input', function (e) {
                            // 给相应的data属性赋值，进而触发该属性的set方法
                            vm[name] = e.target.value;
                        });
                        node.value = vm[name]; // 将data的值赋给该node
                        node.removeAttribute('v-model');
                    }
                };
            }
            // 节点类型为text
            if (node.nodeType === 3) {
                if (reg.test(node.nodeValue)) {
                    var name = RegExp.$1; // 获取匹配到的字符串
                    name = name.trim();
                    // node.nodeValue = vm[name]; // 将data的值赋给该node
                    new Watcher(vm, node, name);        
                }
            }
        }
        function Watcher (vm, node, name) {
            Dep.target = this;    
            this.name = name;
            this.node = node;
            this.vm = vm;
            this.update();
            Dep.target = null;
        }
	    Watcher.prototype = {
	        update: function () {
	            this.get();
	            this.node.nodeValue = this.value;
	        },
	        // 获取data中的属性值
	        get: function () {
	            this.value = this.vm[this.name]; // 触发相应属性的get
	        }
	    }
	    function Dep () {
	        this.subs = []
	    }
	    Dep.prototype = {
	        addSub: function(sub) {
	            this.subs.push(sub);
	        },
	        notify: function() {
	            this.subs.forEach(function(sub) {
	                sub.update();
	            });
	        }
	    };
        function Vue (options) {
            this.data = options.data;
            var data = this.data;
            observe(data, this);
            var id = options.el;
            var dom = nodeToFragment(document.getElementById(id), this);
            // 编译完成后，将dom返回到app中
            document.getElementById(id).appendChild(dom); 
        }
        var vm = new Vue({
            el: 'app',
            data: {
                text: 'hello world'
            }
        });
    </script>
</body>
</html>
```

![img](https://box.kancloud.cn/3dd36cae8a7a50e31425254155b53b3b_624x346.png)

> **Observer 数据监听器**，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者，内部采用Object.defineProperty的getter和setter来实现。
> **Compile 指令解析器**，它的作用对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。
> **Watcher 订阅者**， 作为连接 Observer 和 Compile 的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数。
> **Dep 消息订阅器**，内部维护了一个数组，用来收集订阅者（Watcher），数据变动触发notify 函数，再调用订阅者的 update 方法。

从图中可以看出，当执行 new Vue() 时，Vue 就进入了初始化阶段，一方面Vue 会遍历 data 选项中的属性，并用 Object.defineProperty 将它们转为 getter/setter，实现数据变化监听功能；另一方面，Vue 的指令编译器Compile 对元素节点的指令进行扫描和解析，初始化视图，并订阅Watcher 来更新视图， 此时Wather 会将自己添加到消息订阅器中(Dep),初始化完毕。

当数据发生变化时，Observer 中的 setter 方法被触发，setter 会立即调用Dep.notify()，Dep 开始遍历所有的订阅者，并调用订阅者的 update 方法，订阅者收到通知后对视图进行相应的更新。

### 双向绑定示例

MVVM模式本身是实现了双向绑定的，在Vue.js中可以使用v-model指令在表单元素上创建双向数据绑定。

```
<!--这是我们的View-->
<div id="app">
    <p>{{ message }}</p>
    <input type="text" v-model="message"/>
</div>
```

> 代码示例如下：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>it研习社-基于Vue实现的 数据双向绑定</title>
		<style type="text/css">
			#demo{
				width: 800px;
				margin: 200px auto;
			}
			input{
				width: 600px;
				height: 50px;
				border:10px solid green;
				padding-left: 10px;
				font:30px/50px "微软雅黑";
			}
			.msg{
				width: 600px;
				font:30px/50px "微软雅黑";
				color:red;
			}
		</style>
		<script src="vue.js"></script>
		<script type="text/javascript">
			window.onload=function(){
				new Vue({
				el: '#demo',
				data: {
				   msg:'welcome vue.js',
				}
			})
			}
		</script>
	</head>
	<body>
		<div id="demo">
			<input v-model='msg'/>
			<div class="msg">
				{{msg}}
			</div>
		</div>
	</body>
</html>
```

将message绑定到文本框，当更改文本框的值时，{{ message }} 中的内容也会被更新。

![img](https://box.kancloud.cn/3dd7204e47bdcfebbc88e75e2a40d7f2_449x212.gif)

反过来，如果改变message的值，文本框的值也会被更新，我们可以在Chrome控制台进行尝试。
![img](https://box.kancloud.cn/184bf84402dd679852660846f51df5f0_678x596.gif)

Vue实例的data属性指向exampleData，它是一个引用类型，改变了exampleData对象的属性，同时也会影响Vue实例的data属性。

接下来，我们来做一个关于数据绑定的小练习

> 预览：<https://ityanxi.github.io/Vue-tutorial/chapter02/vue-mvvm1.html>

![img](https://box.kancloud.cn/f3fe8c58de408b704ae84c6785e2417f_472x77.png)

> 代码示例如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vue双向数据绑定练习</title>
    <script src="vue.js"></script>
</head>
<body>
    <div id="demo">
        姓：<input type="text" v-model="firstName">
        名：<input type="text" v-model="lastName">
        <hr>
        姓名：{{firstName+lastName}}
    </div>

    <script>
        var vm=new Vue({
            el:"#demo",
            data:{
                firstName:"huang",
                lastName:"jack",
            }
        });
    </script>
</body>
</html>
```


  