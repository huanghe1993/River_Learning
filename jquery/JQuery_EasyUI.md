## [01] JQuery_EasyUI入门

[TOC]

## 一、EasyUI下载和使用

EasyUI官方下载地址：http://www.jeasyui.com/download/index.php，目前最新的版本是：jQuery EasyUI 1.5.5.1

解压之后得到的内容为如下的形式：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180503/ii0330Hm78.png?imageslim)

## 二、EasyUI入门

### 2.1、引入必要的js和css样式文件

要想在页面中使用EasyUI，那么首先要做的就是在页面中引入必要js和css样式文件，以在jsp文件中使用EasyUI为例，文件引入的顺序如下所示：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180503/L6C5KdbG9d.png?imageslim)

创建第一个jquery_easyUI页面：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>easyUI的入门</title>
    <!-- 引入css文件 -->
    <link rel="stylesheet" type="text/css" href="../themes/default/easyui.css">
    <link rel="stylesheet" type="text/css" href="../themes/icon.css">

    <!-- 引入js文件，有顺序限制 -->
    <script type="text/javascript" src="../js/jquery-1.8.2.js"></script>
    <script type="text/javascript" src="../js/jquery.easyui.min.js"></script>

    <!-- 所有的css文件和所有的js文件位置不限 -->
</head>

<body>

    <!--第一：写一个普通的div标签
        第二：提倡为div标签取一个id属性，将来用jquery好定位
        第三：为普通div标签添加easyUI组件的样式 
              所有的easyui组件的样式均按照如下格式进行设置：
              easyui-组件名称
        第四：如果要用easyUI组件自身的属性时，必须在普通标签上书写data-options属性名，
             内容为key:value ,key:value ,如果value是String类型加引号，外面双引号，则里面是单引号 
        注意：要在普通标签中书写data-options属性的前提是：在 普通标签上加class="easyui-组件名"-->

    <!--iconCls：设置图标，icon-save：保存图标
        closable：定义是否显示关闭按钮
        collapsible:定义是否显示可折叠按钮
        minimizable:定义是否显示最小化按钮。
        maximizable:定义是否显示最大化按钮。  
        -->
    <div id="xx"
         class="easyui-panel"
         style="width: 500px;height: 300px; padding: 35px;"
         title="我的面板"
         data-options="iconCls:'icon-save',closable:true,collapsible:true,minimizable:true,maximizable:true">
        这是我的第一个EasyUI程序
    </div>
</body>
</html>
```

显示的结果如下：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180503/ihkBjcFChL.png?imageslim)



