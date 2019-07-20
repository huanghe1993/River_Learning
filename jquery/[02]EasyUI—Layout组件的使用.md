# [02]EasyUI—Layout组件的使用

[TOC]

## 一、Layout布局

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180503/2FbfCB5gmk.png?imageslim)

### 1.1、panel面板

面板作为承载其它内容的容器。这是构建其他组件的基础（比如：layout,tabs,accordion等）。它还提供了折叠、关闭、最大化、最小化和自定义行为。面板可以很容易地嵌入到web页面的任何位置。

1. 通过标签创建面板

通过标签创建更简单。添加'easyui-panel'类ID到`<div/>`标签。有`class="easyui-panel" `和`data-options`

```html
<div id="p" class="easyui-panel" title="My Panel"     
        style="width:500px;height:150px;padding:10px;background:#fafafa;"   
        data-options="iconCls:'icon-save',closable:true,    
                collapsible:true,minimizable:true,maximizable:true">   
    <p>panel content.</p>   
    <p>panel content.</p>   
</div>  
```

2. 创建面板程序

让我们创建面板右上角的的工具栏。

调用方法的语法：`$('selector').plugin('method', parameter); `,注意这个plugin是调用的插件

```HTML

    <div id="p" style="padding: 10px;">
        <p>panel content</p>
        <p>panel content</p>
    </div>

    <script>
        $("#p").panel({
            width:500,
            height:150,
            title: 'My Panel',
            tools: [{
                iconCls:'icon-add',
                handler:function(){alert('new')}
            },{
                iconCls:'icon-save',
                handler:function(){alert('save')}
            }]
        });
    </script>
```

> 在测试的过程中遇到了一个问题，就是在调用move方法的时候，位置没有任何变化，发现需要在data-options中加上`style:{position:'absolute',left:450,top:250}`的属性,使用绝对定位就可以解决问题

调用panel的move方法：

```html
<div id="p" style="padding: 25px;" data-options="style:{position:'absolute'}"> >
        <p>panel content</p>
        <p>panel content</p>
</div>
<script>
      $(function () {
          $('#p').panel('move', {
              left: 250,
              top: 250
          });
      })
    </script>
```

调用panel的关闭窗口的事件：

```html
    <script>
        $(function () {
            $('#p').panel({
                onClose:function () {
                    alert("关闭面板事件触发");
                }
            })
        })
    </script>
```

最后的运行结果：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180503/kJGefJCK8f.png?imageslim)
**读取内容**
当加载成功的时候让我们通过ajax加载面板内容并显示一些消息。


```html
<script>
    $(function () {
        $('#p').panel({
            href: '../json/city.json',
            onLoad: function () {
                alert('loaded successfully');
            }
        });
    })
</script>
```
![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180503/JdC5ILila6.png?imageslim)

### 1.2、tab选项卡

选项卡显示一批面板。但在同一个时间只会显示一个面板。每个选项卡面板都有头标题和一些小的按钮工具菜单，包括关闭按钮和其他自定义按钮。

#### 1.2.1、创建面板

1. 通过标签创建选项卡

通过标签可以更容易的创建选项卡，我们不需要写任何Javascript代码。只需要给<div/>标签添加一个类ID`'easyui-tabs'`。每个选项卡面板都通过子`<div/>`标签进行创建，用法和panel(面板)相同。

```html
<div id="tt" class="easyui-tabs" style="width:500px;height:250px;">   
    <div title="Tab1" style="padding:20px;display:none;">   
        tab1    
    </div>
  	//closable:是否可以关闭
  	//style:overflow:auto表示的是多余的内容裁剪
  	//display:none表示的是此元素不会被显示
    <div title="Tab2" data-options="closable:true" style="overflow:auto;padding:20px;display:none;">   
        tab2    
    </div>   
    <div title="Tab3" data-options="iconCls:'icon-reload',closable:true" style="padding:20px;display:none;">   
        tab3    
    </div>   
</div> 
```

2. 通过Javascript创建选项卡

下面的代码演示如何使用Javascript创建选项卡，当该选项卡被选择时将会触发'onSelect'事件（个人感觉并不是创建而是事件的触发）。

```javascript
$('#tt').tabs({    
    border:false,    
    onSelect:function(title){    
        alert(title+' is selected');    
    }    
}); 
```

#### 1.2.2、添加新的选项卡面板

添加一个新的包含小工具菜单的选项卡面板，小工具菜单图标(8x8)被放置在关闭按钮之前。这个新开的tab都是会打开的

```HTML
<script>
    $(function(){
        $('#tt').tabs('add', {
            title: 'New Tab',
            content: 'Tab Body',
            closable: true,
            tools: [{
                iconCls: 'icon-reload',
                handler: function () {
                    alert('refresh');
                }
            }]
        }); 

    })
</script>
```