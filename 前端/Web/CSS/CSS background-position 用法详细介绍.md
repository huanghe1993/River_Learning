# CSS background-position 用法详细介绍

**语法：** 

> background-position : length || length 
> background-position : position || position 

**取值：** 

> length  : 百分数 | 由浮点数字和单位标识符组成的长度值。请参阅 长度单位 
> position  : top | center | bottom | left | center | right 

**说明：** 

> - 设置或检索对象的背景图像位置。必须先指定 background-image 属性。
> - 该属性定位不受对象的补丁属性( padding )设置影响。
> - 默认值为： 0% 0% 。此时背景图片将被定位于对象不包括补丁( padding )的内容区域的左上角。
> - 如果只指定了一个值，该值将用于横坐标。纵坐标将默认为 50% 。如果指定了两个值，第二个值将用于纵坐标。
> - 如果设置值为 right center ，因为 right 作为横坐标值将会覆盖 center 值，所以背景图片将被居右定位。
> - 对应的脚本特性为 backgroundPosition。

**注：**

 本文中使用的图片大小为 300px*120px，为了能很清晰的表达图形的哪部分被隐藏了，按照图片的大小平均分成了9等份。同时背景图片容器区域绘制出绿色边框清晰显示容器的范围。

**1、background-position:0 0;**
背景图片的左上角将与容器元素的左上角对齐。该设置与background-position:left top;或者background-position:0% 0%;设置的效果是一致的。例如：

```css
    .container{
        width:300px;
        height:150px;
        background:transparent url(bg.jpg) no-repeat scroll 0 0;
        border:5px solid green;
    }
```

效果如下图1：

![img](https://pic002.cnblogs.com/images/2010/139631/2010110312090476.gif)                                     



**2、该属性定位不受对象的补丁属性( padding )设置影响。**

例如，我们给容器元素增加padding值，背景图片的左上角还是与容器元素的左上角对齐。在此处只是影响到了容器元素的高度和宽度。

```css
 .container{
        width:300px;
        height:150px;
        background:transparent url(bg.jpg) no-repeat scroll 0 0;
        border:5px solid green;
        padding:50px;
    }
```

效果如图2：

![img](https://pic002.cnblogs.com/images/2010/139631/2010110312091698.gif)                                                

**3、background-position:-70px -40px;**

图片以容器左上角为参考向左偏移70px，向上偏移 40px，示例：

```css
 .container{
        width:300px;
        height:150px;
        background:transparent url(bg.jpg) no-repeat scroll -70px -40px;
        border:5px solid green;
    }

```

效果如图3：

![img](https://pic002.cnblogs.com/images/2010/139631/2010110312093583.gif)                                  图 3

**4、background-position:70px 40px;**

图片以容器左上角为参考向右偏移70px，向下偏移 40px，示例：

```css
.container{
        width:300px;
        height:150px;
        background:transparent url(bg.jpg) no-repeat scroll 70px 40px;
        border:5px solid green;
    } 
```

效果如图4：

![img](https://pic002.cnblogs.com/images/2010/139631/2010110312094765.gif)                                   图 4

**5、background-position:50% 50%;**

图片水平和垂直居中。与 background-position:center center;效果等同。

等同于x：`{容器(container)的宽度—背景图片的宽度}*x百分比`，超出的部分隐藏。
等同于y：`{容器(container)的高度—背景图片的高度}*y百分比`，超出的部分隐藏。

 例如：

```css
 .container{
        width:300px;
        height:150px;
        background:transparent url(bg.jpg) no-repeat scroll 50% 50%;
        border:5px solid green;
    }
```

其x=(300-210)*50%=45px;

y=(150-120)*50%=15px;

效果如图5：

![img](https://pic002.cnblogs.com/images/2010/139631/2010110312095970.gif)                                     图 5

由于超出部分别往两端延伸，所以我们可以先制作一张宽度足够宽图片设置水平值为50%，这样可以用来适应不同的浏览器，使得图片水平充满浏览器窗口并且居中。替代margin:50 auto的功能。

**6、background-position:-50% -50%;**

等同于x：-{容器(container)的宽度—背景图片的宽度}*x百分比，超出的部分隐藏。
等同于y：-{容器(container)的高度—背景图片的高度}*y百分比，超出的部分隐藏。

```css
   .container{
        width:300px;
        height:150px;
        background:transparent url(bg.jpg) no-repeat scroll -50% -50%;
        border:5px solid green;
   }
```

效果如图6： 

![img](https://pic002.cnblogs.com/images/2010/139631/2010110312101125.gif)                                



**7、background-position:100% 100%;**

图片处于容器元素的右下角，与 background-position:right bottom;效果等同。

示例：

```css
  .container{
        width:300px;
        height:150px;
        background:transparent url(bg.jpg) no-repeat scroll 100% 100%;
        border:5px solid green;
    }
```

效果如图7：

![img](https://pic002.cnblogs.com/images/2010/139631/2010110312102122.gif)                                   

 