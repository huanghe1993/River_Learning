# BOM

[TOC]



## BOM的概念

BOM(Browser Object Model) 是指浏览器对象模型，浏览器对象模型提供了独立于内容的、可以与浏览器窗口进行互动的对象结构。BOM由多个对象组成，其中代表浏览器窗口的Window对象是BOM的顶层对象，其他对象都是该对象的子对象。

我们在浏览器中的一些操作都可以使用BOM的方式进行编程处理，

比如：刷新浏览器、后退、前进、在浏览器中输入URL等

## BOM的顶级对象window

window是浏览器的顶级对象，当调用window下的属性和方法时，可以省略window
注意：window下一个特殊的属性 window.name,是字符串类型的

## 对话框

- alert()
- prompt()：提示用户输入相应的内容，点击确定的时候，获取到输入的内容
- confirm()：确认对话框，返回的是boolean类型的

## 页面加载事件

- onload

```javascript
window.onload = function () {
  // 当页面加载完成执行
  // 当页面完全加载所有内容（包括图像、脚本文件、CSS 文件等）执行
}
```

- onunload

```javascript
window.onunload = function () {
  // 当用户退出页面时执行
}
```

## 定时器

#### setTimeout()和clearTimeout()

在指定的毫秒数到达之后执行指定的函数，只执行一次

```javascript
// 创建一个定时器，1000毫秒后执行，返回定时器的标示
var timerId = setTimeout(function () {
  console.log('Hello World');
}, 1000);

// 取消定时器的执行
clearTimeout(timerId);
```

#### setInterval()和clearInterval()

定时调用的函数，可以按照给定的时间(单位毫秒)周期调用函数

```javascript
// 创建一个定时器，每隔1秒调用一次
var timerId = setInterval(function () {
  var date = new Date();
  console.log(date.toLocaleTimeString());
}, 1000);

// 取消定时器的执行
clearInterval(timerId);
```

案例：

```
定时器
简单动画
```

定时器代码：

```html
<!--
 * @Description: 
 * @Author: river
 * @LastEditors: huanghe
 * @Date: 2019-04-01 11:59:17
 * @LastEditTime: 2019-04-01 17:37:34
 -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .time-item {
            width: 430px;
            height: 45px;
            margin: 0 auto;
        }

        .time-item strong {
            background: orange;
            color: #fff;
            line-height: 49px;
            font-size: 36px;
            font-family: Arial;
            padding: 0 10px;
            margin-right: 10px;
            border-radius: 5px;
            box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.2);
        }

        .time-item>span {
            float: left;
            line-height: 49px;
            color: orange;
            font-size: 32px;
            margin: 0 10px;
            font-family: Arial, Helvetica, sans-serif;
        }

        .title {
            width: 260px;
            height: 50px;
            margin: 200px auto 50px auto;
        }
    </style>
</head>

<body>
    <h1 class="title">距离生日，还有</h1>

    <div class="time-item">
        <span><span id='day'>00</span>天</span>
        <strong><span id='hour'>00</span>时</strong>
        <strong><span id='minute'>00</span>分</strong>
        <strong><span id='second'>00</span>秒</strong>

    </div>


    <script>
        //每一秒种计算一次
        //目标时间
        var endDate = new Date('2019-7-15');
        var spanDay = document.getElementById('day');
        var spanHour = document.getElementById('hour');
        var spanMinute = document.getElementById('minute');
        var spanSecond = document.getElementById('second');

        setInterval(function () {
            //当前时间
            var startDate = new Date();
            var interval = getInterval(startDate, endDate);
            spanDay.innerText = interval.day;
            spanHour.innerText = interval.hour;
            spanMinute.innerText = interval.minute;
            spanSecond.innerText = interval.second;

        }, 1000);

        function getInterval(start, end) {
            //日期的毫秒数
            var interval = end - start;
            //求 天，小时，分钟，秒
            var day, hour, minute, second;

            // 日期相差的秒数
            interval /= 1000;

            second = Math.round(interval % 60);
            minute = Math.round(interval / 60 % 60);
            hour = Math.round(interval / 60 / 60 % 24);
            day = Math.round(interval / 60 / 60 / 24);

            return {
                day: day,
                hour: hour,
                minute: minute,
                second: second
            }
        }
    </script>
</body>

</html>
```



### location对象

location对象是window对象下的一个属性，使用的时候可以省略window对象

location可以获取或者设置浏览器地址栏的URL

#### location有哪些成员？

- 使用chrome的控制台查看

- 查MDN

  [MDN](https://developer.mozilla.org/zh-CN/)

- 成员

  - assign()/reload()/replace()
  - hash/host/hostname/search/href……

#### URL

统一资源定位符 (Uniform Resource Locator, URL)

- URL的组成

```
scheme://host:port/path?query#fragment
http://www.itheima.com:80/a/b/index.html?name=zs&age=18#bottom
scheme:通信协议
	常用的http,ftp,maito等
host:主机
	服务器(计算机)域名系统 (DNS) 主机名或 IP 地址。
port:端口号
	整数，可选，省略时使用方案的默认端口，如http的默认端口为80。
path:路径
	由零或多个'/'符号隔开的字符串，一般用来表示主机上的一个目录或文件地址。
query:查询
	可选，用于给动态网页传递参数，可有多个参数，用'&'符号隔开，每个参数的名和值用'='符号隔开。例如：name=zs
fragment:信息片断
	字符串，锚点.
```

#### 作业

解析URL中的query，并返回对象的形式

```javascript
function getQuery(queryStr) {
  var query = {};
  if (queryStr.indexOf('?') > -1) {
    var index = queryStr.indexOf('?');
    queryStr = queryStr.substr(index + 1);
    var array = queryStr.split('&');
    for (var i = 0; i < array.length; i++) {
      var tmpArr = array[i].split('=');
      if (tmpArr.length === 2) {
        query[tmpArr[0]] = tmpArr[1];
      }
    }
  }
  return query;
}
console.log(getQuery(location.search));
console.log(getQuery(location.href));
```

### history对象

- back()
- forward()
- go()

### navigator对象

- userAgent

## 特效

### 偏移量

- offsetParent用于获取定位的父级元素
- offsetParent和parentNode的区别

```javascript
var box = document.getElementById('box');
console.log(box.offsetParent);
console.log(box.offsetLeft);
console.log(box.offsetTop);
console.log(box.offsetWidth);
console.log(box.offsetHeight);
```

![1498743216279](F:/%E5%89%8D%E7%AB%AF/%E5%B7%A5%E5%85%B7/05%20WebAPI/05%20WebAPI/%E7%AC%94%E8%AE%B0+%E6%BA%90%E4%BB%A3%E7%A0%81/WebAPI%E8%B5%84%E6%96%99/WebAPI%E8%B5%84%E6%96%99/05-WebAPI07%E5%92%8C08/%E7%AC%94%E8%AE%B0/media/1498743216279.png)

### 客户区大小

```javascript
var box = document.getElementById('box');
console.log(box.clientLeft);
console.log(box.clientTop);
console.log(box.clientWidth);
console.log(box.clientHeight);
```

![1504075813134](F:/%E5%89%8D%E7%AB%AF/%E5%B7%A5%E5%85%B7/05%20WebAPI/05%20WebAPI/%E7%AC%94%E8%AE%B0+%E6%BA%90%E4%BB%A3%E7%A0%81/WebAPI%E8%B5%84%E6%96%99/WebAPI%E8%B5%84%E6%96%99/05-WebAPI07%E5%92%8C08/%E7%AC%94%E8%AE%B0/media/1504075813134.png)

### 滚动偏移

```javascript
var box = document.getElementById('box');
console.log(box.scrollLeft)
console.log(box.scrollTop)
console.log(box.scrollWidth)
console.log(box.scrollHeight)
```

![1498743288621](F:/%E5%89%8D%E7%AB%AF/%E5%B7%A5%E5%85%B7/05%20WebAPI/05%20WebAPI/%E7%AC%94%E8%AE%B0+%E6%BA%90%E4%BB%A3%E7%A0%81/WebAPI%E8%B5%84%E6%96%99/WebAPI%E8%B5%84%E6%96%99/05-WebAPI07%E5%92%8C08/%E7%AC%94%E8%AE%B0/media/1498743288621.png)

### 案例 

- 拖拽案例
- 弹出登录窗口
- 放大镜案例
- 模拟滚动条
- 匀速动画函数
- 变速动画函数
- 无缝轮播图
- 回到顶部  

## 附录

### 元素的类型

![1497169919418](F:/%E5%89%8D%E7%AB%AF/%E5%B7%A5%E5%85%B7/05%20WebAPI/05%20WebAPI/%E7%AC%94%E8%AE%B0+%E6%BA%90%E4%BB%A3%E7%A0%81/WebAPI%E8%B5%84%E6%96%99/WebAPI%E8%B5%84%E6%96%99/05-WebAPI07%E5%92%8C08/%E7%AC%94%E8%AE%B0/media/1497169919418.png)