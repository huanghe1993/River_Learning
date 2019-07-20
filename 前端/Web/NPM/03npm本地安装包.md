# 本地包安装、更新和卸载

[TOC]



有两种方式用来安装 npm 包：本地安装和全局安装。至于选择哪种方式来安装，取决于我们如何使用这个包。

- 如果你自己的模块依赖于某个包，并通过 Node.js 的 `require` 加载，那么你应该选择本地安装，这种方式也是 `npm install` 命令的默认行为。
- 如果你想将包作为一个命令行工具，（比如 grunt CLI(命令行界面)），那么你应该选择全局安装。

想要了解更多关于 `install` 命令行的行为，可以查看 [CLI 文档](https://www.npmjs.cn/cli/install)。

## 安装一个包

### 安装

可以使用下面的命令来安装一个包：

```
> npm install <package_name>
```

上述命令执行之后将会在当前的目录下创建一个 `node_modules` 的目录（如果不存在的话），然后将下载的包保存到这个目录下。

### 测试：

为了确认 `npm install` 是正常工作的，可以检查 `node_modules` 目录是否存在，并且里面是否含有你安装的包的文件夹。

### 实例：

安装一个叫做 `lodash` 的包。安装成功之后，如果 `node_modules` 目录下存在一个名为 `lodash` 的文件夹，则说明成功安装了这个包。

**Microsoft Windows:**

```powershell
C:\ npm install lodash
C:\ dir node_modules

#=> lodash
```

### 哪个版本的包会被安装了？

在本地目录中如果没有 `package.json` 这个文件的话，那么最新版本的包会被安装。

如果存在 `package.json` 文件，则会在 `package.json` 文件中查找针对这个包所约定的[语义化版本规则](https://www.npmjs.cn/getting-started/semantic-versioning)，然后安装符合此规则的最新版本。

### 使用已安装的包

一旦将包安装到 `node_modules` 目录中，你就可以使用它了。比如在你所创建的 Node.js 模块中，你可以 `require` 这个包。

### 实例：

创建一个名为 `index.js` 的文件，并保存如下代码：

```js
// index.js
var lodash = require('lodash');
 
var output = lodash.without([1, 2, 3], 1);
console.log(output);
```

运行 `node index.js` 命令。应当输出 `[2, 3]`。

如果你没能正确安装 `lodash`，你将会看到如下的错误信息：

```
module.js:340
    throw err;
          ^
Error: Cannot find module 'lodash'
```

可以在 `index.js` 所在的目录中运行 `npm install lodash` 命令来修复这个问题。

## 更新本地安装的包

定期更新你的应用所依赖的包（package）是个好习惯。因为依赖包的开发者更新了代码，你的应用也就能够获得提升。

为了完成这个任务需要：

1. 在 `package.json` 文件所在的目录中执行 `npm update` 命令。
2. 执行 `npm outdated` 命令。不应该有任何输出。

## 卸载本地安装的包

如需删除 node_modules 目录下面的包（package），请执行：

`npm uninstall <package>`:

```
npm uninstall lodash
```

如需从 `package.json` 文件中删除依赖，需要在命令后添加参数 `--save`:

```
npm uninstall --save lodash
```

注意：如果你将安装的包作为 "devDependency"（也就是通过 `--save-dev` 参数保存的），那么 `--save` 无法将其从 `package.json` 文件中删除。所以必须通过 `--save-dev` 参数可以将其卸载。