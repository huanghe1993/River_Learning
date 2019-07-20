# NPM （node.js package management）

[TOC]



## 一、npm 简介

- **npm**：一个包管理工具
- 包/模块：为了JavaScript编写人员<strong>共享</strong>他们为解决一些特殊问题所编写的代码和给其他编写人员在他们自己的项目中<strong>重用</strong>这些代码，并且<strong>实时更新</strong>。这些可重用的代码叫做 <strong>包 或者 模块</strong>
  
- 一个包是一个目录，里面有一个或者多个文件
- 一个典型的应用例如一个网站，依赖于成千上万的包。这些包通常比较小。一般的思想是用比较好的解决方法解决了一个小问题，用这些小问题组成一个更大的问题的解决方案。
- [官方网站](https://www.npmjs.com/ )



## 二、npm包安装

npm是用Node.js编写的，所以你需要安装Node.js才能使用npm。您可以通过Node.js网站安装npm，也可以安装Node Version Manager或NVM

### 方式一：从 Node.js 网站安装 npm

**1. 安装 Node.js 和 npm**

如果您使用的是OS X或Windows，请使用[Node.js下载页面](<https://nodejs.org/en/download/>)中的一个安装程序。请务必安装标有LTS的版本。其他版本尚未经过npm测试。

![mark](http://man.hhaxmm.cn/blog/20190606/SwbqEibl87bY.png)

安装完成后，运行`node -v`。可以查看安装node的版本。

### 方式二：使用NVM安装node.js和npm

**1、安装nvm**

在[nvm的下载页面](<https://github.com/coreybutler/nvm-windows/releases>)下载windows版本或者是Linux版本的安装包，有两种版本`nvm-noinstall.zip`(便携版)和`nvm-setup.zip`(exe安装版)，两者唯一区别就是便携版需要手动配置全局变量，而安装版只需要在安装时选定安装目录则会自动配置好。

![mark](http://man.hhaxmm.cn/blog/20190610/0u0qcvf5X2Th.png)

**2、nvm-setup.zip解压，安装。**

直接双击安装，可以使用默认的选项。也可以自己选择安装地址。然后安装过程中会自动把路径写入到全局变量。

> 注意： 如果安装了杀毒软件，应该先关闭杀毒软件，因为写入全局变量是一个敏感操作，某些杀毒软件会报警（不关闭，报警时需要选择允许操作）

![mark](http://man.hhaxmm.cn/blog/20190610/KbkmrvTlvvyS.png)



**3、便携版安装**

- 下载最新版的`nvm-noinstall.zip`后解压放到`C:\dev\nvm`(可以放到任意位置，此处是我安装的目录，注意文件夹名不能存在空格);

![mark](http://man.hhaxmm.cn/blog/20190610/H0RYEWocWUHw.png)

- 配置`nvm`，生成`settings.txt`，填写配置

　　方法一：双击`install.cmd`，会生成`settings.txt`文件（生成位置就是你输入的地址，一般是在`nvm`目录下，如果不是，需要拷贝过来）
　　方法二：直接在nvm目录下新建`settings.txt`文件

```powershell
root: C:\dev\nvm
path: C:\dev\nodejs
arch: 64
proxy: none
node_mirror: http://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

 - setting.txt的配置文件说明：
   
   - root ： nvm的存放地址
   
   - path ： 存放指向node版本的快捷方式，使用nvm的过程中会自动生成。一般写的时候与nvm同级。
   - arch ： 电脑系统是64位就写64,32位就写32
- proxy ： 代理
   - node_mirror: node镜像源，安装node时会从此镜像源下载。
   - npm_mirror: 同上，npm镜像源
   
- 全局变量配置
  1.添加变量`NVM_HOME`，值为`C:\dev\nvm`
  2.添加变量`NVM_SYMLINK`，值为`D:\dev\nodejs`
  3.添加变量`NVM_HOME`和`NVM_SYMLINK`到全局变量`path`: 修改`path`的值最后加上`;%NVM_HOME%;%NVM_SYMLINK%;`
  到此便携版nvm安装完成

**3、安装node.js**

```powershell
# 查看所有可以安装的node.js版本
nvm list available
# 下载
nvm install 8.2.1
# 使用
nvm use 8.2.1
# 安装最新的node
nvm install latest
```

**4、使用nvm管理node和npm**

```powershell
#查看安装的node&npm
nvm list

#切换node版本
nvm use [version]

# 卸载某个版本node
nvm uninstall [version]
```

## 三、npm更新

**2.更新 npm**

安装node.js时，会自动安装npm。但是，npm比Node.js更频繁地更新，因此请确保您拥有最新版本。请运行：

```powershell
npm -v
```

更新

```powershell
npm install -g npm
```

如果您看到的版本与最新版本不匹配，请运行：

```powershell
npm install npm @ latest -g
```

这将安装最新的官方测试版本的npm。要安装将来发布的版本，请运行：

```powershell
npm install npm@next -g
```

### 



## 三、解决npm安装包被墙的问题

- --registry
  - npm config set registry=https//registry.npm.taobao.org 
- cnpm
  - 淘宝NPM镜像,与官方NPM的同步频率目前为10分钟一次 
  - 官网: http://npm.taobao.org/ 
  - npm install -g cnpm –registry=https//registry.npm.taobao.org 
  - 使用cnpm安装包: cnpm install 包名

### npm常用命令

- 安装包
  - 全局安装：npm install -g 包名称
  - 本地安装：npm install  包名称
  - 指定版本安装：npm install -g 包名称@版本号
- 更新包
- 卸载包
  - npm uninstall 包名称

### yarn基本使用

- 类比npm基本使用

## 自定义包

### 包的规范

- package.json必须在包的顶层目录下
- 二进制文件应该在bin目录下
- JavaScript代码应该在lib目录下
- 文档应该在doc目录下
- 单元测试应该在test目录下

### package.json字段分析

- name：包的名称，必须是唯一的，由小写英文字母、数字和下划线组成，不能包含空格
- description：包的简要说明
- version：符合语义化版本识别规范的版本字符串
- keywords：关键字数组，通常用于搜索
- maintainers：维护者数组，每个元素要包含name、email（可选）、web（可选）字段
- contributors：贡献者数组，格式与maintainers相同。包的作者应该是贡献者数组的第一- 个元素
- bugs：提交bug的地址，可以是网站或者电子邮件地址
- licenses：许可证数组，每个元素要包含type（许可证名称）和url（链接到许可证文本的- 地址）字段
- repositories：仓库托管地址数组，每个元素要包含type（仓库类型，如git）、url（仓- 库的地址）和path（相对于仓库的路径，可选）字段
- dependencies：生产环境包的依赖，一个关联数组，由包的名称和版本号组成
- devDependencies：开发环境包的依赖，一个关联数组，由包的名称和版本号组成

### 自定义包案例