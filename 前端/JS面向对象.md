# JS面向对象

## （1）JS中对象的基本的定义的方法

–JS是基于对象（不是面向对象的）：无法创建自定义的类型、不能很好的支持继承和多态。基于对象的语言JavaScript

```javascript
var hero=new Object();

//定义对象的属性
hero.money=100000;
hero.level=6;

//定义对象具有的方法
hero.attack=function(){
	console.log("攻击");
}

console.log(hero);
console.log(hero.money);
console.log(hero.level);
hero.attack();
```

## （2）JS中this和new的使用

```javascript
谁调用this就是谁
1、
function test() {
console.log(this);
}
test();  //window.test();
//上面的this是window，实际是window调用test()
2、
p1.sayHi(); 
//sayHi()中的this，是p1，此时是p1调用sayHi()
3、
构造函数中的this，始终是new的当前对象
```

this关键字的特点：

- this只出现在函数中

- ```
  p1.sayHi(); 
  //sayHi()中的this，是p1，此时是p1调用sayHi()
  ```

  谁调用函数，this就是指的是谁

- new People();   People中的this代指被创建的对象的实例

new关键字的使用：

- 在内存中开辟一块空间，存储创建出来的对象
- 把this设置成为当前的对象
- 执行内部代码，设置对象的属性和方法
- 返回新创建的对象

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    var aaa = new stu();
    console.log(aaa);
    aaa.say();

    function stu(){
        this.say = function () { //
            console.log(this);
        }
    }

</script>
</body>
</html>
```



## (3)构造函数,创建对象

–new后面调用函数，我们称为构造函数。Object()
我们把他视为一个构造函数，构造函数的本质就是一个函数，只不过构造函数的目的是为了创建新对象，为新对象进行初始化(设置对象的属性)

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    //需求：多个自定义对象。
    //缺点：代码冗余，方式比较low。当我们创建空白对象的时候：new Object();
    //利用构造函数自定义对象。

    var stu1 = new Student("王五");
    var stu2 = new Student("赵六");

    console.log(stu1);
    stu1.sayHi();、//sayHi()中的this是stu1，此时是stu1调用sayHi()

    console.log(stu2);
    stu2.sayHi();

//    console.log(typeof stu1);
//    console.log(typeof stu2);

    //创建一个构造函数
    function Student(name){
        //构造函数中的对象指的是this。也就是说的是new关键字把this设置为当前的对象，this代指的是被创			建的对象的实例。
        this.name = name;
        this.sayHi = function () {
            console.log(this.name+"说：大家好！");
        }
    }

</script>
</body>
</html>
```

## (4)特殊的属性的绑定的方式

对象名.属性

对象名[变量]   对象名[值]

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    var stu = new Object();
    var aaa = "age";

    //对象名.属性
    stu.name = "拴柱";
//    stu.aaa = 19;
    //对象名[变量]   对象名[值]
    stu[aaa] = 20;
    stu[0] = "你好";

    console.log(stu);

//    var　arr = [1,2,3];
//    arr[0] = 4;


</script>
</body>
</html>
```

属性绑定的第二种方式是：

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>

</head>
<body>
    <div>111</div>

<script>
    var div = document.getElementsByTagName("div")[0];

    div.onclick = fn;

    function fn() {
        alert(1);
    }

    //第二种属性绑定
//    div["onclick"] = function () {
//        alert(2);
//    }

//    console.log(div.onclick);
//    console.log(div.onmouseover);

    div.removeEventListener("click",fn);


</script>
</body>
</html>
```

