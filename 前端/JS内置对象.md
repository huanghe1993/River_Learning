# JS内置对象

## （1）Array

### (1.1)常用的方法

- 检测数组

–instanceof      比如 if(arr1  instanceof  Array)用于判断是否符合这个对象

–Array.isArray()  //HTML5中新增

- 转换数组

–toString()  //把数组转换成字符串，每一项用,分割

–valueOf()  //返回数组对象本身

- join

 ```html
  <!DOCTYPE html>
  <html>
  <head lang="en">
      <meta charset="UTF-8">
      <title></title>
  </head>
  <body>
  <script>

      //定义方法
  //    var arr1 = [1,2,3];
  //    var arr2 = new Array(1,2);
  //
  //    var str1 = new String("abc");
  //    var str2 = "abc";

  //    console.log(arr1 instanceof Array);
  //    console.log(Array.isArray(arr2));
  //    console.log(Array.isArray(str1));
  //    console.log(str1 instanceof String);
  //    console.log(str2 instanceof String);

  //    var stu = new Stu();
  //    function Stu(){
  //
  //    }
  //    console.log(stu instanceof Stu);

      //数组转换
  //    console.log(arr1);
  //    console.log(arr1.toString());  //数组转化为字符串并且以逗号的形式进行分割
  //    console.log(typeof arr1.toString());
  //    console.log(arr1.valueOf());
  //    console.log(typeof arr1.valueOf());

      //join是把数组元素用特殊方式链接成字符串(参数决定用什么链接，无参默认用逗号链接)
  //    var arr = ["关羽","张飞","刘备"];
  //    var str1 = arr.join();
  //    var str2 = arr.join(" ");//如果用空格的话，那么元素之间会有一个空格
  //    var str3 = arr.join("");//用空字符串，链接元素，无缝连接
  //    var str4 = arr.join("&");
  //    console.log(str1);
  //    console.log(str2);
  //    console.log(str3);
  //    console.log(str4);

  </script>
  </body>
  </html>
 ```
## （1.2）拓展的数组的API(arguments的使用)

- 栈操作(先进后出)

–push()     //在数组的最末尾添加元素（返回新数组的长度）

–pop() //取出数组中的最后一项，修改length属性

- 队列操作(先进先出)

–shift()//取出数组中的第一个元素，返回被删除项

–unshift() //在数组最前面插入项，返回数组的长度

- 排序方法

–reverse()  //翻转数组

–sort(); //即使是数组sort也是根据字符，从小到大排序

```javascript
 //问题：只能通过第一位排列。解决问题办法：设计的时候就是这么设计的，可以通过回掉函数进行规则设置。
var arr2 = [7,6,15,4,13,2,1];
//    console.log(arr);
//    console.log(arr.sort());
    console.log(arr2);
    console.log(arr2.sort(function (a,b) {
        return b-a;
    }));
```

–带参数的sort是如何实现的？

```javascript
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

//    var arr = [];
//    arr.push(1);
//    console.log(arr);

    fn(1,2);
    fn(1,2,3);
    fn(1,2,3,4,5);
    function fn(a,b){
        //只在函数中使用，实参的数组。
       arguments[0] = 0;
       console.log(arguments);
        //伪数组：不能修改长短的数组。(可以修改元素，但是不能变长变短)
       arguments.push(1);
       console.log(arguments instanceof Array);

       //形参个数
       console.log(fn.length);
       //实参个数
       console.log(arguments.length);

        //arguments.callee整个函数。函数名。
//        console.log(arguments.callee);
    }

</script>
</body>
</html>
```

## （1.3）数组的截取和替换

•操作方法

–concat() //把参数拼接到当前数组

–slice() //从当前数组中截取一个新的数组，不影响原来的数组，参数start从0开始,end从1开始

–splice()//删除或替换当前数组的某些项目，参数start,deleteCount,options(要替换的项目)

•位置方法

–indexOf()、lastIndexOf()   //如果没找到返回-1

•迭代方法不会修改原数组

–every()、filter()、forEach()、map()、some()

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    var arr1 = ["a","b","c"];
    var arr2 = [1,2,3];

    //concat把数组链接在一起
    var arr3 = arr1.concat(arr2);
    console.log(arr1);
    console.log(arr2);
    console.log(arr3);

    //slice数组的截取
//    var arr4 = arr3.slice(2);["c",1,2,3]
//    var arr4 = arr3.slice(-2);[2,3]
//    var arr4 = arr3.slice(4,2);//[]
//    var arr4 = arr3.slice(2,4);//["c", 1]索引值包括坐标的不包括右边的。
//    console.log(arr3);
//    console.log(arr4);

    //splice操作和截取原数组
    var arr5 = ["关羽","关羽","关羽"];
    //替换的元素不能是以数组形式存在，否则将整个数组放进原数组中。
    var arr4 = arr3.splice(0,3,"关羽","关羽","关羽");
    console.log(arr3);
    console.log(arr4);

</script>
</body>
</html><!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    var arr1 = ["a","b","c"];
    var arr2 = [1,2,3];

    //concat把数组链接在一起
    var arr3 = arr1.concat(arr2);
    console.log(arr1);
    console.log(arr2);
    console.log(arr3);

    //slice数组的截取
//    var arr4 = arr3.slice(2);["c",1,2,3]
//    var arr4 = arr3.slice(-2);[2,3]
//    var arr4 = arr3.slice(4,2);//[]
//    var arr4 = arr3.slice(2,4);//["c", 1]索引值包括坐标的不包括右边的。
//    console.log(arr3);
//    console.log(arr4);

    //splice操作和截取原数组
    var arr5 = ["关羽","关羽","关羽"];
    //替换的元素不能是以数组形式存在，否则将整个数组放进原数组中。后面的3是截取多少个
    var arr4 = arr3.splice(0,3,"关羽","关羽","关羽");
    console.log(arr3);
    console.log(arr4);

</script>
</body>
</html>
```

## （1.4）数组的查找

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>
    var arr = ["a","b","c","d","c","b","a"];

    //indexOf从前往后给元素查索引，找到一个后，立刻返回。
    console.log(arr.indexOf("a"));
    console.log(arr.indexOf("b"));
    console.log(arr.indexOf("x"));

    //lastIndexOf从后往前给元素查索引，找到一个后，立刻返回。
    console.log(arr.lastIndexOf("a"));
    console.log(arr.lastIndexOf("b"));
    console.log(arr.lastIndexOf("x"));
</script>
</body>
</html>
```

## (1.5)数组的遍历


```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    var arr = ["关长","张飞","赵龙","马超","黄忠"];

    //every()他的返回值是一个boolean类型值。而参数是一个回调函数。
//    var bool = arr.every(function (element,index,array) {
////        element = "aaa";
////        array[index] = "aaa";
////        console.log(element);
////        console.log(index);
////        console.log(array);
////        return true;
//        if(element.length>2){
//            return false;
//        }
//        return true;
//    });
//
//    alert(bool);

    //filter返回值是一个新数组。return为true的数组。
//    var arr1 = arr.filter(function (ele,index,array) {
//        if(ele.length>2){
//            return true;
//        }
//        return false;
//    });
//
//    console.log(arr1);


    //foreach遍历数组(无返回值，纯操作数组中的元素)
//    var str = "";
//    arr.forEach(function (ele,index,array) {
//        str+=ele;
//    });
//    alert(str);


    //map有返回值，返回什么都添加到新数组中。
//    var arr2 = arr.map(function (ele,index,array) {
//        return ele+"你好";
//    })
//
//    console.log(arr2);


    //some有返回值，函数结果有一个是true，本方法结果也是true。
//    var flag = arr.some(function (ele,index,array) {
//        if(ele.length>2){
//            return true;
//        }
//        return false;
//    })
//
//    alert(flag);


</script>
</body>
</html>
```
## （1.6）清空数组

```javascript
var array = [1,2,3,4,5,6];
方式1
 array.splice(0,array.length); //删除数组中所有项目 
方式2
array.length = 0; //length属性可以赋值，其它语言中length是只读
方式3
array = [];  //推荐

```