# Node -01

## 1. Node.js的诞生

[TOC]



![mark](http://man.hhaxmm.cn/blog/20190512/pWrstuOVGpYY.png)

- 作者Ryan Dahl 瑞恩·达尔
    + 2004 纽约 读数学博士 
    + 2006 退学到智利 转向开发 
    + 2009.5对外宣布node项目，年底js大会发表演讲 
    + 2010 加入Joyent云计算公司 
    + 2012 退居幕后

> Node.js 是一种建立在Google Chrome’s v8 engine上的 non-blocking (非阻塞）, event-driven （基于事件的） I/O平台. 
> Node.js平台使用的开发语言是JavaScript，平台提供了操作系统低层的API，方便做服务器端编程，具体包括文件操作、进程操作、通信操作等系统模块

Node.js可以用来做什么？

- 具有复杂逻辑的动态网站 
- WebSocket服务器 
- 命令行工具 
- 带有图形界面的本地应用程序 
- ......

## 2. 终端基本使用

### 2.1 打开应用

- notepad 打开记事本
- mspaint 打开画图
- calc 打开计算机
- write 写字板
- sysdm.cpl 打开环境变量设置窗口
### 2.2 常用命令
- md 创建目录
- rmdir(rd) 删除目录，目录内没有文档。rd /s/q 删除带文件的目录及其子目录
- rm 删除文件
- echo on a.txt 创建空文件
- echo    123 >a.txt    向文件中写内容
- echo    456>> a.txt  追加内容
- del 删除文件
- rm 文件名 删除文件
- cat 文件名 查看文件内容
- cat > 文件名 向文件中写上内容。

## 3. Node.js开发环境准备

1. 普通安装方式[官方网站](https://nodejs.org/zh-cn/)

2. 多版本安装方式
    - 卸载已有的Node.js
    - 下载[nvm](https://github.com/coreybutler/nvm-windows)
    - 在C盘创建目录dev
    - 在dev目中中创建两个子目录nvm和nodejs
    - 并且把nvm包解压进去nvm目录中
    - 在install.cmd文件上面右键选择【以管理员身份运行】
    - 打开的cmd窗口直接回车会生成一个settings.txt文件，修改文件中配置信息
    
    ```txt
    root:  C:\dev\nvm
    path:  
    arch: 64 
    proxy: none
    ```
    
    - 配置nvm和Node.js环境变量
        + NVM_HOME:C:\dev\nvm
        + NVM_SYMLINK:C:\dev\nodejs
    - 把配置好的两个环境变量加到Path中

**nvm常用的命令**

- nvm list 查看当前安装的Node.js所有版本
- nvm install 版本号 安装指定版本的Node.js
- nvm uninstall 版本号 卸载指定版本的Node.js
- nvm use 版本号 选择指定版本的Node.js

## 4. Node.js之HelloWorld
- 命令行方式REPL(read-eval-print-loop) 读取代码-执行-打印-循环

<img src='http://man.hhaxmm.cn/blog/20190512/quql1Dv03cGK.png' align='left' style=' width:500px;height:300 px'/>

- 运行文件方式

<img src='http://man.hhaxmm.cn/blog/20190512/zLxHeIi96QxL.png' align='left' style=' width:500px;height:300 px'/>

- 全局对象概览

<https://nodejs.org/dist/latest-v10.x/docs/api/globals.html>

简单的举例

```js
//文件的全路径 
console.log(__filename);  //D:\Code\web\front-end\node_learning\02.js
// 文件路径
console.log(__dirname);   //D:\Code\web\front-end\node_learning

// 定时函数
// 创建一个定时器，1000毫秒后执行，返回定时器的标示
var timerId = setTimeout(function () {
    console.log('Hello World');
}, 1000);

// 取消定时器的执行
clearTimeout(timerId);

// argv是一个数组，第一项数据是node.js环境的路径，第二项是当前执行js文件的全路径
// 从第三个参数开始表示命令行的参数
// [ 'C:\\dev\\nodejs\\node.exe',
//  'D:\\Code\\web\\front-end\\node_learning\\02.js' ]
console.log(process.argv);
```



## 5. 服务器端模块化
- 服务器端模块化规范CommonJS与实现Node.js
- 模块导出与引入

```js
var sum = function (a,b) {
    return parseInt(a)+parseInt(b);
}

// require导出模块成员
// exports.sum =sum;

// module导出成员的另外一种方式
module.exports =sum;
```

```js
// 引入模块
var module = require('./03.js');

// var ret = module.sum(12,13);
// module引入模块
var ret = module(12,25);

console.log(ret);
```



- 模块导出机制分析
- 模块加载规则
    + 模块查找 不加扩展名的时候会按照如下后缀顺序进行查找 .js .json .node
- 模块分类
    + 自定义模块
    + 系统核心模块
        * fs 文件操作
        * http 网络操作
        * path 路径操作
        * querystring 查询参数解析
        * url url解析
        * ......

> 模块化相关规则：
>
> 1、如何定义模块：一个JS文件就是一个模块，模块内的成员是相互独立的
>
> 2、模块成员的导出和引入

## 6. ES6常用语法
- 变量声明let与const

```js
/*
 * @Description: 
 * @Author: river
 * @Date: 2019-05-12 18:16:59
 * @LastEditTime: 2019-05-12 18:30:44
 * @LastEditors: huanghe
 */
// 声明变量let和const
// console.log(flag);

// 1、let声明的变量不存在预解析
// let flag =124;
// ---------------
// 2、let声明的变量在同一个作用域内不允许重复
// let flag=123;
// let flag=456;
// console.log(flag);
// -----------------
// 3、ES6中声明了块级别的作用域
// 块内部声明的变量，在外部是不可以访问的（包括for循环里面的变量）
// if (true) {
//     // var flag=1;
//     let flag =1;
// }
// console.log(flag);
// for (let index = 0; index < array.length; index++) {
//     // for循环括号中声明的变量只能在循环体的内部使用
//     console.log(index);
//    
// }
// --------------------
// 4、在块级作用域内部，变量只能先声明后使用
// if (true) {
//     console.log(flag);
//     let flag=123;
// }


// =============const===============
// 1、const用来声明常量（值不发生改变）,声明的常量必须初始化
// const n=1;
// n=2;  
```



- 变量的解构赋值
    + 数组解构赋值
    + 对象解构赋值
    + 字符串解构赋值

```js
/*
 * @Description: 
 * @Author: river
 * @Date: 2019-05-12 18:33:23
 * @LastEditTime: 2019-05-12 18:44:23
 * @LastEditors: huanghe
 */
/* 变量的解构赋值 */

var a = 1;
var b = 2;
var c = 3;

var a =1 ,b=2,c=3;

// ES6的变量的解构赋值 -----数组的解构赋值
// let [a,b,c] = [1,2,3];
// console.log(a,b,c);

// ES6的变量的解构赋值------对象的解构赋值
// let {foo,bar} = {foo : 'hello',bar : 'hi'};
// let {foo,bar} = {bar : 'hi',foo : 'hello'};

// 对象属性别名(如果有了别名，那么原来的名字就无效了)
// let {foo:abc,bar} = {bar : 'hi',foo : 'nihao'};
// console.log(foo,bar);

// 对象的解构赋值指定默认值
// let {foo:abc='hello',bar} = {bar : 'hi'};
// console.log(abc,bar);

// let {cos,sin,random} = Math;
// console.log(typeof cos);
// console.log(typeof sin);
// console.log(typeof random);
// --------------------------------------
// 字符串的解构赋值
// let [a,b,c,d,e,length] = "hello";
// console.log(a,b,c,d,e);
// console.log(length);

// console.log("hello".length);

// let {length} = "hi";
// console.log(length);


```



- 字符串扩展
    + includes()
    + startsWith()
    + endsWith()
    + 模板字符串

```js
/*
    字符串相关扩展
    includes() 判断字符串中是否包含指定的字串（有的话返回true，否则返回false）
               参数一：匹配的字串；参数二：从第几个开始匹配
    startsWith()  判断字符串是否以特定的字串开始
    endsWith()  判断字符串是否以特定的字串结束

    模板字符串
*/
// console.log('hello world'.includes('world',7));

// let url = 'admin/index.php';
// console.log(url.startsWith('aadmin'));
// console.log(url.endsWith('phph'));

// ----------------------------------
let obj = {
    username : 'lisi',
    age : '12',
    gender : 'male'
}

let tag = '<div><span>'+obj.username+'</span><span>'+obj.age+'</span><span>'+obj.gender+'</span></div>';
console.log(tag);
// 反引号表示模板，模板中的内容可以有格式，通过${}方式填充数据
let fn = function(info){
    return info;
}
let tpl = `
    <div>
        <span>${obj.username}</span>
        <span>${obj.age}</span>
        <span>${obj.gender}</span>
        <span>${1+1}</span>
        <span>${fn('nihao')}</span>
    </div>
`;
console.log(tpl);
```



- 函数扩展
    + 参数默认值
    + 参数结构赋值
    + rest参数
    + 扩展运算符
    + 箭头函数

```js
/*
    函数扩展
    1、参数默认值
    2、参数解构赋值
    3、rest参数
    4、...扩展运算符
*/

// 1、参数默认值
// function foo(param){
//     let p = param || 'hello';
//     console.log(p);
// }
// foo('hi');
// -----------------ES6的写法
// function foo(param = 'nihao'){
//     console.log(param);
// }
// foo('hello kitty');
// ----------------------------------
// function foo(uname='lisi',age=12){
//     console.log(uname,age);
// }
// // foo('zhangsan',13);
// foo();
// 2、参数解构赋值
// function foo({uname='lisi',age=13}={}){
//     console.log(uname,age);
// }
// foo({uname:'zhangsan',age:15});
// --------------------------------------
// 3、rest参数（剩余参数），其余的参数作为数组
// function foo(a,b,...param){
//     console.log(a);
//     console.log(b);
//     console.log(param);
// }
// foo(1,2,3,4,5);

// 扩展运算符 ...
function foo(a,b,c,d,e,f,g){
    console.log(a + b + c + d + e + f + g);
}
// foo(1,2,3,4,5);
let arr = [1,2,3,4,5,6,7];
// foo.apply(null,arr); 之前的方法
foo(...arr);

// 合并数组
let arr1 = [1,2,3];
let arr2 = [4,5,6];
let arr3 = [...arr1,...arr2];
console.log(arr3);
```

箭头函数

```js
/*
    箭头函数
*/
// function foo(){
//     console.log('hello');
// }
// foo();

// let foo = () => console.log('hello');
// foo();

// function foo(v){
//     return v;
// }
// let foo = v => v;
// let ret = foo(111);
// console.log(ret);

// 多个参数必须用小括号包住
// let foo = (a,b) => {let c = 1; console.log(a + b + c);}
// foo(1,2);

// let arr = [123,456,789];
// arr.forEach(function(element,index){
//     console.log(element,index);
// });
// arr.forEach((element,index)=>{
//     console.log(element,index);
// });

// 箭头函数的注意事项：
// 1、箭头函数中this取决于函数的定义，而不是调用
// function foo(){
//     // 使用call调用foo时，这里的this其实就是call的第一个参数
//     // console.log(this);
//     setTimeout(()=>{
//         console.log(this.num);
//     },100);
// }
// foo.call({num:1});
// ----------------------------------
// 2、箭头函数不可以new
// let foo = () => { this.num = 123;};
// new foo();
// ------------------------------------
// 3、箭头函数不可以使用arguments获取参数列表，可以使用rest参数代替
// let foo = (a,b) => {
//     // console.log(a,b);
//     console.log(arguments);//这种方式获取不到实参列表
// }
// foo(123,456);
let foo = (...param) => {
    console.log(param);
}
foo(123,456 );
```



- 类与继承