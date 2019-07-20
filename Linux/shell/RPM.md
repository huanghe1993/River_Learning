## RPM

在 Linux 操作系统下，几乎所有的软件均通过RPM 进行安装、卸载及管理等操作。RPM 的全称为Redhat Package Manager ，是由Redhat 公司提出的，用于管理Linux 下软件包的软件。Linux 安装时，除了几个核心模块以外，其余几乎所有的模块均通过RPM 完成安装。RPM 有五种操作模式，分别为：安装、卸载、升级、查询和验证。

### 语法 

```shell
rpm(选项)(参数)
```

### 选项 

```
-a：查询所有套件；
-b<完成阶段><套件档>+或-t <完成阶段><套件档>+：设置包装套件的完成阶段，并指定套件档的文件名称；
-c：只列出组态配置文件，本参数需配合"-l"参数使用；
-d：只列出文本文件，本参数需配合"-l"参数使用；
-e<套件档>或--erase<套件档>：删除指定的套件；
-f<文件>+：查询拥有指定文件的套件；
-h或--hash：套件安装时列出标记；
-i：显示套件的相关信息；
-i<套件档>或--install<套件档>：安装指定的套件档；
-l：显示套件的文件列表；
-p<套件档>+：查询指定的RPM套件档；
-q：使用询问模式，当遇到任何问题时，rpm指令会先询问用户；
-R：显示套件的关联性信息；
-s：显示文件状态，本参数需配合"-l"参数使用；
-U<套件档>或--upgrade<套件档>：升级指定的套件档；
-v：显示指令执行过程；
-vv：详细显示指令执行过程，便于排错。
```

### 参数 

软件包：指定要操纵的rpm软件包。

### 实例 

**如何安装rpm软件包**

rpm软件包的安装可以使用程序rpm来完成。执行下面的命令：

```
rpm -ivh your-package.rpm
```

其中your-package.rpm是你要安装的rpm包的文件名，一般置于当前目录下。

安装过程中可能出现下面的警告或者提示：

```
... conflict with ...
```

可能是要安装的包里有一些文件可能会覆盖现有的文件，缺省时这样的情况下是无法正确安装的可以用`rpm --force -i`强制安装即可

```
... is needed by ...
... is not installed ...
```

此包需要的一些软件你没有安装可以用`rpm --nodeps -i`来忽略此信息，也就是说`rpm -i --force --nodeps`可以忽略所有依赖关系和文件问题，什么包都能安装上，但这种强制安装的软件包不能保证完全发挥功能。