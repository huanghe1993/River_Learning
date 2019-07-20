# html初始2

## （1）过渡版本的html和严格类型的html的差别

在过渡型的html中，html是想过渡到xml但是没有完成这个过渡；比如在过渡的html中<br>是可以的，但是在严格的类型中这样写是不行的，必须得加上<br/>

## (2)现在发展到h5

!+tab就可以生产html的标签结构

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">    //规定了网页的编码格式
  	<meta name="keywords" content="java培训，Android培训">  //网页的关键字(给搜索引擎的优化看的)
  	<meta name="description" content="java学习"> //网页的描述（显示在搜索页面上的东西）
    <meta http-equive="refresh" content="5;http://www.itcast.com"> //网页重定向，5秒之后进行跳转（作用：比如我们在搜索www.xiaomi.com的时候会自动的进行跳转到www.mi.com）
  	<link rel="stylesheet" href="1.css">//连接外部样式表
  	<link rel="icon" href="favicon.ico"> //设置icon
	<title>Document</title>   //文件的标题
</head>
<body>
	
</body>
</html>
```

```html
Ascll ：能识别的字符很少
Ansi  ：
Unicode  ：可以识别英文，日文。。。编码 
Gbk  ：可以识别中文编码
Gb2312 ：可以识别中文编码
Big5	:繁体字符的编码
Utf-8   通用字符集

```

## (3)表格

**table>tr*3>td*3**可以生成3行3列的表格

```html
<table>    表格
<tr>       行
<td></td> 列
<td></td>
<td></td>
</tr>
</table>

```

| border="1"                  | 边框                           |
| --------------------------- | ---------------------------- |
| width=“500”                 | 宽度                           |
| height=“300”                | 高度                           |
| cellspacing="2"             | 单元格和单元格之间的距离                 |
| cellpadding="2"             | 内容距边框的距离                     |
| align="left"\|right\|center | 如果直接给表格用align=”center”  表格居中 |
|                             | 如果给tr或者td使用   ，tr或者td内容居中    |
|                             | bgcolor=”red”    背景颜色        |

## （4）表格的标准结构

```html
<table>
<thead>   //表頭
<tr>
<td></td>
<td></td>
</tr>
</thead>
<tbody>  //表身
<tr>
<td></td>
<td></td>
</tr>
</tbody>

  
<tfoot>//表脚
<tr>
<td></td>
<td></td>
</tr>
</tfoot>
</table>

```

## （5）表头和单元格的合并

>表头 <caption></caption>
>
>colspan=”2”  合并同一行上的单元格
>
>rowspan=”2”  合并同一列上的单元格

## （6）表格的标题、边框颜色、内容垂直对齐

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title></title>
</head>
<body>
	<table border="2" width="500" height="300" bordercolor="red" >
		<tr>
			<th>张三</th>
			<th>20</th>
			<th>前端</th>
		</tr>
		<tr>
			<td>张三</td>
			<td>20</td>
			<td>前端</td>
		</tr>
		<tr>
			<td>张三</td>
			<td>20</td>
			<td>前端</td>
		</tr>
	</table>
</body>
</html>
```

在使用th之后内容会自动的加粗和对齐：标题的文字自动加粗水平居中对齐

bordercolor="red“设置边框的颜色

<td valign="top">张三</td>表示的是内容垂直居中  ，默认的是middle

  Valign=”top   | middle   |  bottom”

## （7）细线表格

cellspacing="0" 单元格和单元格之间的距离

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>细线表格</title>
</head>
<body>
	<table  width="500" height="300" bgcolor="green" cellspacing="1">
		<tr  bgcolor="white">
			<td></td>
			<td></td>
			<td></td>
			<td></td>
		</tr>
		<tr  bgcolor="white">
			<td></td>
			<td></td>
			<td></td>
			<td></td>
		</tr>
		<tr  bgcolor="white">
			<td></td>
			<td></td>
			<td></td>
			<td></td>
		</tr>
		<tr  bgcolor="white">
			<td></td>
			<td></td>
			<td></td>
			<td></td>
		</tr>
	</table>
</body>
</html>
```

## (8)表单

表单的作用是收集信息

表单的组成：提示信息+表单的控件

- 文本输入框

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>表单</title>
</head>
<body>
	<form action="1.php" method="get">
		用户名：<input type="text" name="username"><br>
		密  码：<input type="password" name="pwd"><br>
		<input type="submit">
	</form>
</body>
</html>
```



| action              | 提交的信息传给后台，处理信息         |
| ------------------- | ---------------------- |
| method              | get 和post的方式           |
| type="text"         | 提交的是文本域，文本输入框          |
| maxlength="6"       | 控制输入的最大的长度             |
| readonly="true"     | 只读，不能编辑                |
| disabled="disabled" | 未激活，也是不能编辑的            |
| name                | 为了分辨是哪个控件传输过来的（输入框的名称） |
| value="前端"          | value是默认的值             |

- 密码输入框

|      | type="password"        |
| ---- | ---------------------- |
|      | 文本输入框的所有的属性对密码输入框都是有效的 |
|      |                        |
|      |                        |

- 单选框

| type="radio"  | 单选按钮                      |
| ------------- | ------------------------- |
| name="gender" | 输入框的名称                    |
| checked       | 设置的默认的选中项                 |
| 注意：           | 只有将name的值设置相同的时候，才能实现单选效果 |

- 下拉列表

| <select><option>湖北</option></select> |             |
| ------------------------------------ | ----------- |
| selected 或者selected=selected         | 设置的是默认的选中项  |
| Multiple=”multiple”                  | 将下拉列表设置为多选项 |
| <optgroup></optgroup>                | 对下拉列表进行分组   |
| Label=””                             | 分组名称        |

- 多行文本框

| <textarea cols="130" rows="10"></textarea> |      |
| ---------------------------------------- | ---- |
| cols设置的是输入字符的长度                          |      |
| Rows控制输入的行数                              |      |
|                                          |      |

- 文件上传控件

| <input type="file"> |      |
| ------------------- | ---- |
|                     |      |
|                     |      |
|                     |      |

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>表单</title>
</head>
<body>
	<form action="1.php" method="get">
		<fieldset>
			<legend>分组信息</legend>
		用户名：<input type="text" maxlength="6" name="username" value="前端"><br>
		密  码：<input type="password" name="pwd"><br>
		<input type="radio" name="gender" checked="checked">男
		<input type="radio" name="gender">女
		<br>
		省(市):<select>
				<option>湖北</option>
				<option>山西</option>
				<option selected="selected">陕西</option>
			   </select>
		市(区)：<select>
			<optgroup label="北京市">
				<option>海定区</option>
				<option>朝阳区</option>
				<option>大兴区</option>
			</optgroup>
			<optgroup label="广州市">
				<option>越秀区</option>
				<option>天河区</option>
				<option>番禺区</option>
			</optgroup>
		</select>

		<br>
		<!-- 多选框 -->
		<input type="checkbox" checked="checked" name="">语文
		<input type="checkbox" name="">数学
		<input type="checkbox" name="">英语
		<br>

		<!-- 多行文本 -->
		<textarea cols="130" rows="10"></textarea>
		<br><br>

		<!-- 文件上传控件 -->
		<input type="file" name="">
		<br><br>

		<!-- 文件提交 -->
		<input type="submit"><br>

		<!-- 普通的按钮，不能实现提交，配合jS使用 -->
		<input type="button" name="" value="提交">
		<br><br>

		<!-- 图片按钮也是可以实现提交的功能 -->
		<input type="image" name="" src="按钮.jpg">
		<br><br>

		<!-- 重置按钮 把信息回归到默认的状态-->
		<input type="reset" name="">
		</fieldset>
	</form>
</body>
</html>
```

## (9)h5的表单控件

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<form action="1.php" method="post">
		<!-- h5的网址提交 -->
		<input type="url" name=""><br><br>

		<!-- h5的日期控件 -->
		<input type="date" name=""><br><br>

		<!-- 时间控件 -->
		<input type="time" name=""><br><br>

		<!-- 邮件控件 -->
		<input type="email" name=""><br><br>

		<!-- 数字控件 -->
		<input type="number" step="2" name=""><br><br>

		<!-- 滑块控件 -->
		<input type="range" step="2" name=""><br><br>



		<input type="submit" name="">
	</form>
</body>
</html>
```

未完待续。。。。。。