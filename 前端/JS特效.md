# JS特效

## 1.JS的四大系列(offset/scroll/client)事件对象/event（事件被触动时，鼠标和键盘的状态）（通过属性控制）

### 1.1、offset系列

•js中有一套方便的获取元素尺寸的办法就是offset家族；

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/171129/2L9eeGIlBL.png?imageslim)



offsetWidth和offsetHight 以及offsetLeft和offsetTop以及offsetParent共同组成了offset家族

#### 1.1.1、offsetWidth()和offsetHight （检测盒子自身宽高+padding+border）

这两个属性，他们绑定在了所有的节点元素上。获取之后，只要调用这两个属性，我们就能够获取元素节点的宽和高。

offset宽/高  =  盒子自身的宽/高 + padding+border；

offsetWidth =width+padding+border；

offsetHeight =Height+padding+border；

#### 1.1.2、offsetLeft和offsetTop  （检测距离父盒子有定位的左/上面的距离）

返回距离上级盒子（带有定位）左边的位置

如果父级都没有定位则以body为准

offsetLeft 从父亲的padding 开始算,父亲的border 不算。

在父盒子有定位的情况下，offsetLeft ==style.left(去掉px)

#### 1.1.3、offsetParent （检测父系盒子中带有定位的父盒子节点）

1、返回改对象的父级 （带有定位）

​          如果当前元素的父级元素没有进行CSS定位 （position为absolute或             relative，fixed），   offsetParent为body。

2、如果当前元素的父级元素中有CSS定位

​       （position为absolute或 relative，fixed），offsetParent取最近的那个父级元素。

#### 1.1.4、offsetLeft和style.left区别

一、最大区别在于offsetLeft可以返回没有定位盒子的距离左侧的位置。 而 style.left不可以

```
如果父系盒子中都没有定位，以body为基准
style.left只能获取行内式，如果没有返回“”，
```

二、offsetTop 返回的是数字，而 style.top 返回的是字符串，除了数字外还带有单位：px。

三、offsetTop 只读，而 style.top 可读写。（只读是获取值，可写是赋值）

```
style.left和style.top可以进行赋值
 offsetLeft和offsetTop只能获取值，只读
```

四、如果没有给 HTML 元素指定过 top 样式，则style.top 返回的是空字符串。

## 2、动画和封装

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        div {
            width: 100px;
            height: 100px;
            background-color: pink;
            position: absolute;
        }
    </style>
</head>
<body>
<button>动画</button>
<div class="box" style="left: 0px"></div>

<script>
    var btn = document.getElementsByTagName("button")[0];
    var div = document.getElementsByTagName("div")[0];
    //闪动
//        btn.onclick = function () {
//            div.style.left = "500px";
//        }

    //匀速运动
    btn.onclick=function(){
        //定时器，每隔一段的时间向右走一些
        setInterval(function(){
//            div.style.left=parseInt(div.style.left)+10+"px";返回的值是NaN获取的值是不可以用的
            //style.left赋值，用offsetLeft获取值。
            //style.left获取值不方便，获取行内式，如果没有事“”；容易出现NaN；
            //offsetLeft获取值特别方便，而且是现成number方便计算。因为他是只读的不能赋值。
            div.style.left=div.offsetLeft+10+"px";
        },300);
    }
</script>
</body>
</html>
```



## 3、缓动动画

Math.ceil()            向上取整

Math.floor()        向下取整

Math.round();        四舍五入

### 3.1、缓动动画原理

```
leader=leader+(target-leader)/10;

盒子位置 = 盒子本身位置+（目标位置-盒子本身位置）/ 10；
```

### 3.2为什么盒子没有到达指定的位置

盒子本身位置     目标位置      步长         已经到达了的位置

0                                     400                 0                                  0

0                                      400                 40                                40

40                                   400                 36                                76

76                                   400                 32.4                            108.4

.........

JS实际运算时会四舍五入取整，然后计算。

396(四舍五入获取)       400                 0.4                     396.4

396(四舍五入获取)       400                 0.4                     396.4

### 3.3、offsetLeft和style.left的值的获取问题

 获取盒子距离左侧具有定位的父盒子的距离（没有的body），四舍五入取整。

Style.left获取的是具体值。（赋值的时候也是直接赋值）



## 4、第二系列scroll

### 4.1、scroll家族的组成

#### 4.1.1ScrollWidth和scrollHeight（不包括border）

检测盒子的宽高。（调用者：节点元素。属性。）

盒子内容的宽高。（如果有内容超出了，显示内容的高度）

IE567可以比盒子小。 IE8+火狐谷歌不能比盒子小

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        div {
            width: 100px;
            height: 100px;
            padding: 10px;
            margin: 10px;
            border: 10px solid #000;
        }
    </style>
</head>
<body>
<div class="box">
    人生自古谁无死，留取丹心照汉青。
    人生自古谁无死，留取丹心照汉青。
    人生自古谁无死，留取丹心照汉青。
    人生自古谁无死，留取丹心照汉青。
    人生自古谁无死，留取丹心照汉青。
    人生自古谁无死，留取丹心照汉青。

</div>

<script>
    var div=document.getElementsByTagName("div")[0];

    //scrollWidth和scrollHeight不包括border和margin
    //scrollWidth = width + padding;
    // 不包括 border和margin;
    console.log(div.scrollWidth);

    //高度有一个特点：如果文字超出了盒子，高度为超出盒子的内容的高。不超出是盒子本身高度
    //IE8以下，不包括IE8;为盒子本身内容的多少。
    console.log(div.scrollHeight);
</script>
</body>
</html>
```

#### 4.1.2、scrollTop和scrollLeft

网页，被浏览器遮挡的头部和左边部分。

被卷去的头部和左边部分。

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/171201/5gclm5JfEH.png?imageslim)

兼容问题

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/171201/kE9JgChaHJ.png?imageslim)







 

