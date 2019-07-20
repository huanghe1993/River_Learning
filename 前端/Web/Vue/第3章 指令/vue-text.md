## v-text

**类型：** string
**用法：** 更新元素的 textContent。如果要更新部分的 textContent ，需要使用 {{ Mustache }} 插值。

> 示例：

```html
<span v-text="msg"></span>
<!-- 和下面的一样 -->
<span>{{msg}}</span>
```

> 代码示例1

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="vue.js"></script>
</head>
<body>
    <div id="demo" v-text="'今年是'+year+'年'+month+'月'"></div>
    <script>
        new Vue({
            el:'#demo',
            data:{
                year:new Date().getFullYear(),
                month:new Date().getMonth()+1
            }
        });
    </script>
</body>
</html>
```

> 预览：<https://huanghe1993.github.io/chapter03/01v-text1.html>

------

> 代码示例2

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="vue.js"></script>
</head>
<body>
    <div id="demo">
                今年是{{year}}年{{month}}月
    </div>
    <script>
        new Vue({
            el:'#demo',
            data:{
                year:new Date().getFullYear(),
                month:new Date().getMonth()+1
            }
        });
    </script>
</body>
</html>
```

> 预览：<<https://huanghe1993.github.io/chapter03/01v-text2.html>>

### 区别v-text与插值表达式

- 默认v-text是没有闪烁问题的
- v-text会覆盖元素中原本的内容，但是插值表达式只会替换自己的占位符，不会清空整个的元素