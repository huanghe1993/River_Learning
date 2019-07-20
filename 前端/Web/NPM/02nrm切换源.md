# nrm切换镜像源

## 什么是nrm

nrm是npm registry（镜像） 管理工具, 能够查看和切换当前使用的registry。解决npm安装包被墙的问题，以及速度慢的问题。顾名思义，就是说nrm是一个管理npm的工具，允许你快速地在 npm 源间切换。

## 安装nrm

在命令行执行命令，全局安装nrm。

```powershell
npm install -g nrm
```

## 使用

执行命令`nrm ls`查看可选的源。

```powershell
nrm ls                                                                                                                                    

*npm ---- https://registry.npmjs.org/
cnpm --- http://r.cnpmjs.org/
taobao - http://registry.npm.taobao.org/
eu ----- http://registry.npmjs.eu/
au ----- http://registry.npmjs.org.au/
sl ----- http://npm.strongloop.com/
nj ----- https://registry.nodejitsu.com/
```

其中，带`*`的是当前使用的源，上面的输出表明当前源是官方源。

## 切换
如果要切换到taobao源，执行命令nrm use taobao。

##  增加
你可以增加定制的源，特别适用于添加企业内部的私有源，执行命令 nrm add `<registry>`  `<url>`，其中reigstry为源名，url为源的路径。

```powershell
nrm add registry http://192.168.10.127:8081/repository/npm-public/
```

## 删除
执行命令`nrm del <registry>`删除对应的源。

## 测试速度
你还可以通过 `nrm test `测试相应源的响应时间。

```powershell
nrm test npm                                                                                                                               
npm ---- 1328ms
```

 