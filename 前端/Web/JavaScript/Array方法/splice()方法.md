## splice()方法

## 实例

数组中添加新元素：

```js
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2,0,"Lemon","Kiwi");
```

*fruits* 输出结果：

``````
Banana,Orange,Lemon,Kiwi,Apple,Mango
``````

## 定义和用法

splice() 方法用于添加或删除数组中的元素。

**注意：**这种方法会改变原始数组。

### 返回值

如果仅删除一个元素，则返回一个元素的数组。 如果未删除任何元素，则返回空数组。

![img](https://www.runoob.com/wp-content/uploads/2013/08/9F673968-EB30-48C7-90F6-8CE14E737BC1.png)

## 浏览器支持

![Internet Explorer](https://www.runoob.com/images/compatible_ie.gif)![Firefox](https://www.runoob.com/images/compatible_firefox.gif)![Opera](https://www.runoob.com/images/compatible_opera.gif)![Google Chrome](https://www.runoob.com/images/compatible_chrome.gif)![Safari](https://www.runoob.com/images/compatible_safari.gif)

所有主要浏览器都支持splice()。

------

## 语法

```js
array.splice(index,howmany,item1,.....,itemX)
```

## 参数 Values

| 参数                  | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| *index*               | 必需。规定从何处添加/删除元素。 该参数是开始插入和（或）删除的数组元素的下标，必须是数字。 |
| *howmany*             | 必需。规定应该删除多少元素。必须是数字，但可以是 "0"。 如果未规定此参数，则删除从 index 开始到原数组结尾的所有元素。 |
| *item1*, ..., *itemX* | 可选。要添加到数组的新元素                                   |

## 返回值

| Type  | 描述                                                         |
| :---- | :----------------------------------------------------------- |
| Array | 如果从 arrayObject 中删除了元素，则返回的是含有被删除的元素的数组。 |

------

## 更多实例

## 实例

移除数组的第三个元素，并在数组第三个位置添加新元素:

```js
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2,1,"Lemon","Kiwi");
```

*fruits* 输出结果：

```html
Banana,Orange,Lemon,Kiwi,Mango
```

## 实例

从第三个位置开始删除数组后的两个元素：

```js
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2,2);
```

*fruits* 输出结果：

```html
Banana,Orange
```