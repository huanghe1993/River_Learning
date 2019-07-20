# avaScript findIndex() 方法

## 实例

获取数组中年龄大于等于 18 的第一个元素索引位置

```js
var ages = [3, 10, 18, 20];   
function checkAdult(age) {     
    return age >= 18;
}   
function myFunction() {     
    document.getElementById("demo").innerHTML = ages.findIndex(checkAdult); 
}
```

*fruits* 输出结果：

```html
2
```

------

## 定义和用法

findIndex() 方法返回传入一个测试条件（函数）符合条件的数组第一个元素位置。

findIndex() 方法为数组中的每个元素都调用一次函数执行：

- 当数组中的元素在测试条件时返回 *true* 时, findIndex() 返回符合条件的元素的索引位置，之后的值不会再调用执行函数。
- 如果没有符合条件的元素返回 -1

**注意:** findIndex() 对于空数组，函数是不会执行的。

**注意:** findIndex() 并没有改变数组的原始值。

------

## 语法

```js
array.findIndex(function(currentValue, index, arr), thisValue)
```

## 参数

| 参数                                | 描述                                                         |
| :---------------------------------- | :----------------------------------------------------------- |
| *function(currentValue, index,arr)* | 必须。数组每个元素需要执行的函数。 函数参数:参数描述*currentValue*必需。当前元素*index*可选。当前元素的索引*arr*可选。当前元素所属的数组对象 |
| *thisValue*                         | 可选。 传递给函数的值一般用 "this" 值。 如果这个参数为空， "undefined" 会传递给 "this" 值 |

## 技术细节

| 返回值：         | 返回符合测试条件的第一个数组元素索引，如果没有符合条件的则返回 -1。 |
| :--------------- | ------------------------------------------------------------ |
| JavaScript 版本: | ECMAScript 6                                                 |

## 更多实例

## 实例

返回符合大于输入框中数字的数组索引：

```js
var ages = [4, 12, 16, 20];
 
function checkAdult(age) {
    return age >= document.getElementById("ageToCheck").value;
}
 
function myFunction() {
    document.getElementById("demo").innerHTML = ages.findIndex(checkAdult);
}
```