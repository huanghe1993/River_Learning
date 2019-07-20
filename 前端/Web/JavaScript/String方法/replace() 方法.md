# replace() 方法

## 实例

在本例中，我们将执行一次替换，当第一个 "Microsoft" 被找到，它就被替换为 "Runoob"：

```js
var str="Visit Microsoft! Visit Microsoft!"; var n=str.replace("Microsoft","Runoob");
```

*n* 输出结果:

```html
Visit Runoob!Visit Microsoft!
```

------

## 定义和用法

replace() 方法用于在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。

如果想了解更多正则表达式教程请查看本站的：[RegExp 教程](http://www.runoob.com/js/js-obj-regexp.html) 和 our [RegExp 对象参考手册](http://www.runoob.com/jsref/jsref-obj-regexp.html).

该方法不会改变原始字符串。

------

## 浏览器支持

![Internet Explorer](http://www.runoob.com/images/compatible_ie.gif)![Firefox](http://www.runoob.com/images/compatible_firefox.gif)![Opera](http://www.runoob.com/images/compatible_opera.gif)![Google Chrome](http://www.runoob.com/images/compatible_chrome.gif)![Safari](http://www.runoob.com/images/compatible_safari.gif)

所有主要浏览器都支持 replace() 方法。

------

## 语法

*string*.replace(*searchvalue,newvalue*)

## 参数值

| 参数          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| *searchvalue* | 必须。**规定子字符串或要替换的模式的 RegExp 对象**。 请注意，如果该值是一个字符串，则将它作为要检索的直接量文本模式，而不是首先被转换为 RegExp 对象。 |
| *newvalue*    | 必需。一个字符串值。规定了替换文本或生成替换文本的函数。     |

## 返回值

| 类型   | 描述                                                         |
| :----- | :----------------------------------------------------------- |
| String | 一个新的字符串，是用 replacement 替换了 regexp 的第一次匹配或所有匹配之后得到的 |

## 更多实例

## 实例

执行一个全局替换:

```js
var str="Mr Blue has a blue house and a blue car"; var n=str.replace(/blue/g,"red");
```

*n* 输出结果:

```js
Mr Blue has a red house and a red car
```

## 实例

执行一个全局替换, 忽略大小写:

```js
var str="Mr Blue has a blue house and a blue car"; var n=str.replace(/blue/gi, "red");
```

*n* 输出结果:

```js
Mr red has a red house and a red car
```

