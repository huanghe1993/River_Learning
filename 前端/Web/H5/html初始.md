# html初始1

## （1）html的标准的结构

```html
< ! doctype html>    声明文档类型
<html>              根标签
<head>             头标签
<title></title>       标题标签
</head>
<body>             主体标签
</body>

</html>

```

## （2）html的快捷键

| Ctrl+shift+d      | 快速复制一行 | Ctrl+shiif+k      | 快速删除一行                                                 |
| ----------------- | ------------ | ----------------- | ------------------------------------------------------------ |
| Ctrl+鼠标左键单击 | 集体输入     | Ctrl+h            | 查找替换                                                     |
| Ctrl+f            | 查找         | Ctrl+/            | 注释                                                         |
| Html:xt+tab       | Html结构代码 | tab               | 补全标签代码                                                 |
| Ctrl+L            | 快速选择一行 | Ctrl+shift+↑（↓） | 快速上移（下移）一行                                         |
| F11               | 全屏         | Ctrl + 回车       | 添加一行空行                                                 |
| alt+shift+1/2     | 布局进行分屏 | Ctrl+D            | 反复按快捷键，即可继续向下同时选中下一个相同的文本进行同时编辑 |
| Ctrl+K+U          | 改为大写     | Ctrl+K+L          | 改为小写                                                     |
| ctrl+/            | 注释标签     |                   |                                                              |
|                   |              |                   |                                                              |

## （3）html基本的标签的使用

| 标签                                   | 用法                          |
| ------------------------------------ | --------------------------- |
| `<hr/>`                              | 水平线                         |
| `<br/>`                              | 换行                          |
| `<p>文本内容</p>`                        | 段落标签，上下自动生成空白行              |
| `h1-h6  取值到h6`                       | 标题标签，取值只能到h6,h1在一个页面中只能出现一次 |
| `<font size="6" color="red"></font>` | 文本内容标签                      |
| `<strong>文本加粗标签</strong>`            | 文本加粗标签                      |
| `<em>文本倾斜标签</em>`                    | 文本倾斜标签                      |
| `<del>删除线标签</del>`                   | 删除线标签                       |
| `<ins>下划线标签</ins>`                   | 下划线标签                       |

## （4）html的图片标签

| `<img src="3.jpg" alt="huanghe" title="god" width="300" height="500"/>` |      |
| ---------------------------------------- | ---- |
| src:表示的是路径（文件和图片（html文档）在同一个目录(文件夹) ，直接写文件名） |      |
| Alt:替换文本，图片不显示的时候显示文字                    |      |
| title:提示文字，鼠标放在图片上显示出来的提示性的文字            |      |
| Width:图片的宽度                              |      |
| Height:图片的长度，百分之百比例显示，如果只更改图片的宽度或者高度，图片等比例缩放 |      |
| 路径：文件和图片（html文档）在同一个目录(文件夹) ，直接写文件名      |      |
| 路径：图片（html文档）在文件在下一级目录里。文件夹名称+/+图片（html）名称 |      |
|                                          |      |
|                                          |      |

## （5）超链接标签

| `<a href="test1.html" title="图片标签" target=“_self”>超链接</a>` |      |
| ---------------------------------------- | ---- |
| href   去往的路径（跳转的页面） 必写属性                 |      |
| title    提示文本   鼠标放到链接上显示的文字             |      |
| target=”_self”    默认值    在自身页面打开（关闭自身页面，打开链接页面） |      |
| Target=”_blank”   打开新页面 （自身页面不关闭，打开一个新的链接页面） |      |
|                                          |      |

## （6）锚链接

| 首先定义一个锚点`<p id="sd">`   ->超链接到锚点`<a href="#sd">回到顶部</a>` |
| ---------------------------------------- |
| 不知道链接到那个页面的时候，用空链                        |
| `<base target="_blank"> `  让所有的超链接都在新窗口打开 |
|                                          |
|                                          |
|                                          |

## （7）特殊的符号

![特殊符号](http://img.blog.csdn.net/20171107172217969?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzQwMzA0Mzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## （8）无序列表

```html
<ul type="circle">

	<li>   </li>

	<li>   </li>

</ul>
```

type=”square”      小方块
Type=”disc”       实心小圆圈
Type=”circle”      空心小圆圈

## (9)有序列表

```html
<ol type="a" start=3>

	<li>  </li>

	<li>  </li>

</ol>
```

type=”1,a,A,i,I”type的值可以为1,a,A,i,I

start=”3”  决定了开始的位置

## （10）自定义列表

```html
<dl>

 <dt></dt>    小标题

 <dd></dd>   解释标题

 <dd></dd>   解释标题

</dl>
```

## （11）音乐标签

`<embed src="1.mp3" hidden="true">`hidden是进行隐藏出现的会话框

## （12）滚动标签

页面自动滚动效果之：`<marquee>...</marquee>`

中间的内容可以为文字，图片，也可以是由程序生成的文字或图片

属性：height  设置高度   width 设置宽度  bgcolor 设置背景颜色

behavior：设定滚动的方式

alternate：表示在两端之间来回滚动。

scroll：表示由一端滚动到另一端，会重复。

slide：  表示由一端滚动到另一端，不会重复。

direction：设置滚动的方向

down  :向下滚动    left:向左滚动    right:向右滚动   up:向上滚动

loop：设置滚动次数   -1一直滚下去

页面背景音乐：`<embed />`

属性：src   设置音乐路径

​          hidden :隐藏播放按钮   true 隐藏    false显示