### 实例

检测数组 *site* 是否包含 runoob :

```js
let site = ['runoob', 'google', 'taobao'];  
site.includes('runoob');  // true    
site.includes('baidu');  // false
```

------

## 定义和用法

includes() 方法用来判断一个数组是否包含一个指定的值，如果是返回 true，否则false。

```js
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
[1, 2, NaN].includes(NaN); // true
```

------

## 语法

```js
arr.includes(searchElement)
arr.includes(searchElement, fromIndex)
```

------

## 参数说明

| 参数            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| *searchElement* | 必须。需要查找的元素值。                                     |
| *fromIndex*     | 可选。从该索引处开始查找 searchElement。如果为负值，则按升序从 array.length + fromIndex 的索引开始搜索。默认为 0。 |

------

## 技术细节

| 返回值：         | 布尔值。如果找到指定值返回 true，否则返回 false。 |
| :--------------- | ------------------------------------------------- |
| JavaScript 版本: | ECMAScript 6                                      |

------

## 更多实例

### fromIndex 大于等于数组长度

如果fromIndex 大于等于数组长度 ，则返回 false 。该数组不会被搜索:

### 实例

```js
var arr = ['a', 'b', 'c'];   
arr.includes('c', 3);   //false 
arr.includes('c', 100); // false
```

### 计算出的索引小于 0

如果 fromIndex 为负值，计算出的索引将作为开始搜索searchElement的位置。如果计算出的索引小于 0，则整个数组都会被搜索。

