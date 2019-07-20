## 实例

列出数组的每个元素：

```js
<button onclick="numbers.forEach(myFunction)">点我</button>
<p id="demo"></p>
 
<script>
demoP = document.getElementById("demo");
var numbers = [4, 9, 16, 25];
 
function myFunction(item, index) {
    demoP.innerHTML = demoP.innerHTML + "index[" + index + "]: " + item + "<br>"; 
}
</script>
```

输出结果：

```html
index[0]: 4
index[1]: 9
index[2]: 16
index[3]: 25
```

## 定义和用法

forEach() 方法用于调用数组的每个元素，并将元素传递给回调函数。

**注意:** forEach() 对于空数组是不会执行回调函数的。

## 语法

```
array.forEach(function(currentValue, index, arr), thisValue)
```

## 参数

| 参数                                 | 描述                                                         |
| :----------------------------------- | :----------------------------------------------------------- |
| *function(currentValue, index, arr)* | 必需。 数组中每个元素需要调用的函数。 函数参数:参数描述*currentValue*必需。当前元素*index*可选。当前元素的索引值。*arr*可选。当前元素所属的数组对象。 |
| *thisValue*                          | 可选。传递给函数的值一般用 "this" 值。 如果这个参数为空， "undefined" 会传递给 "this" 值 |

------

## 技术细节

| 返回值:          | undefined    |
| :--------------- | ------------ |
| JavaScript 版本: | ECMAScript 3 |

## 更多实例

## 实例

计算数组所有元素相加的总和：

```js
<button onclick="numbers.forEach(myFunction)">点我</button>
 
<p>数组元素总和：<span id="demo"></span></p>
 
<script>
var sum = 0;
var numbers = [65, 44, 12, 4];
 
function myFunction(item) {
    sum += item;
    demo.innerHTML = sum;
}
</script>
```



