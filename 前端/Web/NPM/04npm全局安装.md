# 全局包安装、更新和卸载

[TOC]

## 全局安装包

有两种方式用来安装 npm 包：本地安装和全局安装。选用哪种方式来安装，取决于你如何使用这个包。

- 如果你想将其作为一个命令行工具，那么你应该将其安装到全局。这种安装方式后可以让你在任何目录下使用这个包。比如 grunt 就应该以这种方式安装。
- I如果您想依赖自己模块中的软件包，请在本地安装。例如，如果您使用require语句，则可以使用此选项

将包安装到全局，你应该使用 `npm install -g <package>` 命令，例如：

```
npm install -g jshint
```

*小技巧：如果你安装的 npm 是 5.2 或更高版本，可以使用 npx 运行全局安装的包。*

```powershell
npm install -g 包名称@版本号
```



## 更新全局安装的包

要更新全局包，请键入：

```powershell
npm update -g <package>
```

要找出需要更新的软件包，请键入：

```powershell
npm outdated -g --depth=0
```

要更新所有全局软件包，请键入：

```
npm update -g
```

##  卸载全局安装的包

要卸载全局程序包，请键入：

```
npm uninstall -g <package>
```

要卸载名为jshint的软件包，你应该输入：

```
npm uninstall -g jshint
```