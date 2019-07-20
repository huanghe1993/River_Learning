## v-html

**类型：** string
**用法：** 更新元素的 innerHTML 。

> 注意：内容按普通 HTML 插入 - 不会作为 Vue 模板进行编译 。
> 如果试图使用 v-html 组合模板,可以重新考虑是否通过使用组件来替代。

> 在网站上动态渲染任意 HTML 是非常危险的，因为容易导致 XSS 攻击。只在可信内容上使用 v-html，永不用在用户提交的内容上。

------

> 示例

```html
<div v-html="html"></div>
```

> 代码示例1

```html
<div id="demo" v-html="msg"></div>
    <script>
        var app=new Vue({
            el:'#demo',
            data:{
                msg:'<img src="logo.png">'
            }
        });
    </script>
```

> 预览：<<https://huanghe1993.github.io/chapter03/02v-html1.html>>

------

> 代码示例2

```html
    <div id="demo" v-html="msg"></div>
    <script>
        var app=new Vue({
            el:'#demo',
            data:{
                msg:'<h1>Vue从入门到实战</h1>'
            }
        });
    </script>
```

> 预览：<https://huanghe1993.github.io/chapter03/02v-html2.html>

### 区别v-text与v-html

- v-html可以解析html，而v-text与插值表达式是不能进行解析的

