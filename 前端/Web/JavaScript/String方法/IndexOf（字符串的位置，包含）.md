## 实例

查找字符串 "welcome":

```js
var str="Hello world, welcome to the universe.";
var n=str.indexOf("welcome");
```

*n* 输出结果:

```html
13
```

## 定义和用法

indexOf() 方法可返回某个指定的字符串值在字符串中首次出现的位置。

如果没有找到匹配的字符串则返回 -1。

**注意：** indexOf() 方法区分大小写。

**提示：** 同样你可以查看类似方法 [lastIndexOf()](https://www.runoob.com/jsref/jsref-lastindexof.html) 。

## 语法

*string*.indexOf(*searchvalue*,*start*)

## 参数值

| 参数          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| *searchvalue* | 必需。规定需检索的字符串值。                                 |
| *start*       | 可选的整数参数。规定在字符串中开始检索的位置。它的合法取值是 0 到 string Object.length - 1。如省略该参数，则将从字符串的首字符开始检索。 |

## 返回值

| 类型   | 描述                                                         |
| :----- | :----------------------------------------------------------- |
| Number | 查找指定字符串第一次出现的位置，如果没找到匹配的字符串则返回 -1。 |