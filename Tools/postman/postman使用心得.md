# Postman接口测试与管理工具

[TOC]

## 1. Postman的基础功能

![img](https://upload-images.jianshu.io/upload_images/1190574-5755bdd460fef56c.png?imageMogr2/auto-orient/)

## 2.Postman设置环境变量

**问题描述：**
设置环境变量，变化接口访问的地址，例如localhost，换成其他IP地址。

例如：有个接口login

本地访问为：http://localhost:8061/login

测试环境为：http://11.11.11.136:8061/login

可以把IP地址根据环境动态的改变，就需要Postman中的环境变量来解决。

**解决思路：**
1：在Postman中设置两个环境，本地环境、测试环境

2：在两个环境中分别设置变量名为 base_url的变量，存放各自环境的IP地址

Postman 中获取环境变量值的方式为：{{变量名}}，这样就可以获取变量值。

这里base_url的值获取方式就是： {{base_url}}

3：在Postman的URL地址栏中设置 为 {{base_url}}/login

4：切换不同的环境，base_url的值就会发生改变，这样就达到了IP地址切换的效果。

**设置环境变量：**

 **第一步**：图右上角配置按钮

![mark](http://www.hhaxmm.cn/blog/20190105/bqqnU83RnYBG.png)
**第二步：阿里云环境变量**
![mark](http://www.hhaxmm.cn/blog/20190105/IhvI1FUromWb.png)

点击Add添加成功，这里要说明的是上面的136环境，和本地测试环境也都有这个base_url变量，只是值不同，

**就是切换这个环境取不同的base_url值达到环境的切换**。

如下图所示，环境变量添加完毕



## 3.设置全局变量

**背景**
之前我们做过一个案例，就是在cookie设置登录凭证token。但token有时需要改变，而且大量请求都需要用到这个token。每次请求都改掉token显然不切实际，如果使用上一节介绍的环境变量也无法一次性修改所有请求。这里带大家介绍Postman的全局变量功能，本案例把在header上动态设置登录凭证token的值。操作过程与上一节设置环境变量很相似。

 **实战**

1.点右上角的`小齿轮`，进入环境管理面板，在面板点`Globals`进入全局变量面板。

![1546657250325](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1546657250325.png)

2.key填`token`，value填`123456`（填具体token的值），点右下角`Save`保存全局变量。如有多个可以全部填好再保存。

![mark](http://www.hhaxmm.cn/blog/20190105/WwhA7uvUPzkk.png)

3.在`Headers`中添加一个header，key填`token`（接口自行规定），value为`{{token}}`（刚刚在global设置的key）。鼠标移动到上面，会显示具体的value值。也可以点右上角的小眼睛，看所有的全局变量。 

![mark](http://www.hhaxmm.cn/blog/20190105/kVFi6U64Qpkw.png)

> postman解决动态传参变量问题（token）
> 在一般的用户系统中，我们都会使用token来作为用户登陆系统进行操作的令牌，是时时变化的，每一次做登录接口测试时都会变化，一变化我们保存的全局token就失效了，导致我们无法对用户系统中的其他功能进行操作，如果我们可以在每次登录的时候进行时时保存token的值，那我们就不需要每次测试其他接口时就得重新改一遍token值了
> 解决方法就是：
> 1.登录的时候动态获取token的值和account的值（一般系统这两个值是必须的）
> 2.获取的值保存在已设置的全局变量中（替换设置的全局变量中的值）
> 3.在请求响应头中引用我们要获取的值
> 声明：相应数据为json数据 格式如下
>
> ![mark](http://www.hhaxmm.cn/blog/20190105/SDD6N1dcCXfq.png)



>1.在**登录**的时候获取响应体中的值并保存在全局变量中
>
>![1546657565015](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1546657565015.png)
>
>2.设置其他测试接口的请求头内容
>
>​     在headers中设置，作为请求头的信息传到后台中
>
>然后我们在测试其他接口的时候，postman就会获取全局变量中的值，作为请求头中的参数传过去，这样就可以解决动态token测试后台系统的问题了。