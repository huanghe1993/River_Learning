# JS对象的字面量

## （1）对象字面量

对象的字面量就是一个{}，而里面的属性和方法是以冒号形式对应表示的（键值对）

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
<script>
	
	var obj1=new Object();
	console.log(obj1);

	var arr=[];
	//对象的字面量就是一个{}，而里面的属性和方法是以冒号形式对应表示的（键值对）/其中属性加引号也可以不加引号也可以
	var obj2={name:"张三",age:18,sayHi:function(){
		console.log(1);
	}};
	console.log(obj2);
	obj2.sayHi();

</script>
</body>
</html>
```

## (2)JSON

```json
主要的作用是和后台进行数据的传递

什么是JSON
JavaScript Object Notation（JavaScript对象表示形式）
JavaScript的子集
JSON和对象字面量的区别
JSON的属性必须用双引号引号引起来，对象字面量可以省略
var o = {};  对象字面量          {} JSON

      {
            "name" : "zs",
            "age" : 18,
            "sex" : true,
            "sayHi" : function() {
                console.log(this.name);
            }
        };
```

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>
    //json和对象（对象字面量）的区别仅仅在于，json的key键值对中的键必须带有“”；
    var json = {"name":"拴住","age":18,"arr":[1,2,3]};
    //对象本身没有length,所以不能用for循环遍历
    console.log(json.length);

//    for(var i=0;i<json.length;i++){
//        console.log(json[i]);
//    }

</script>
</body>
</html>
```

## (3)Json的遍历的方式For ....in

```json
1.3	For...in...
Var json = {“aaa”: 1,“bbb”: 2,“ccc”: 3,“ddd”: 4}
for(var key in json){
//key代表aaa,bbb.....等
//json[key]代表1,2,3....等
}

```

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<script>

    //对象本身没有length,所以不能用for循环遍历
    //要用for。。。in...循环
   var aaa = {"name":"拴住","age":18,"arr":[1,2,3]};

   for(var k in aaa){
       console.log(k);
       //aaa.k代表aaa这个对象的k属性的值，并不是k对应的变量值的属性。
//        console.log(aaa.k);
       //aaa[k],代表的是aaa这个对象中k这个变量值对应的属性值。
       console.log(aaa[k]);
   }

   var arr = [1,2,3];
   for(var k in arr){
       console.log(arr[k])
   }


 //   制作一个json,然后
   var json = {};
   console.log(json);
   for(var i=1;i<10;i++){
       json[i] = i*10;
   }
   console.log(json);

   for(var k in json){
       console.log(json[k]);
   }

</script>
</body>
</html>
```

## (4)JS的内置对象Array

Date   Array   Math    RegExp   Error    String

