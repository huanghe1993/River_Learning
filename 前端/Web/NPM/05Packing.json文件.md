# 使用package.json

管理本地安装的npm软件包的最好方法是创建一个 package.json 文件。

一个 package.json 文件：

- 列出你的项目依赖的软件包。
- 允许您使用[语义版本规则](https://docs.npmjs.com/getting-started/docs.npmjs.com/getting-started/semantic-versioning)来指定项目可以使用的包的版本。
- 使您的构建可重复使用，因此与其他开发人员共享更容易。

## 要求

package.json必须具有：

- “名称”

  - 全部小写
  - 一个单词，没有空格
  - 允许破折号和下划线

- “版本号”

  - 以 x.x.x 的形式
  - 遵循 [semver 规范](https://docs.npmjs.com/getting-started/docs.npmjs.com/getting-started/semantic-versioning)

  例如：

  ```
  {
    "name": "my-awesome-package",
    "version": "1.0.0"
  }
  ```

  ## 创建一个 package.json 文件

  有两种基本的方法来创建一个 package.json 文件。

  ### 1.运行 CLI 问卷

  要使用您提供的值创建 package.json，请运行：

  ```powershell
  npm init
  ```

  这将启动一个命令行调查问卷，最终将在您启动命令的目录中创建一个 package.json 。

  ### 2.创建一个默认的 package.json

  要获得默认的 package.json 文件，请使用 --yes 或 -y 标志运行 `npm init`：

  ```
   npm init --yes
  ```

  此方法将使用从当前目录中提取的信息生成默认的 package.json。

  ```powershell
  > npm init --yes
  Wrote to /home/ag_dubs/my_package/package.json:
  
  {
    "name": "my_package",
    "description": "",
    "version": "1.0.0",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "repository": {
      "type": "git",
      "url": "https://github.com/ashleygwilliams/my_package.git"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "bugs": {
      "url": "https://github.com/ashleygwilliams/my_package/issues"
    },
    "homepage": "https://github.com/ashleygwilliams/my_package"
  }
  ```

  - name: 当前目录名称
  - version: 总是1.0.0
  - description: 自述文件中的信息，或空字符串“”
  - main: 总是index.js
  - scripts: 默认情况下创建一个空的测试脚本
  - keywords: 空
  - author: 空
  - license: ISC
  - bugs: 来自当前目录的信息（如果存在）
  - homepage: 来自当前目录的信息（如果存在）

  您还可以为 init 命令设置多个配置选项。 一些有用的：

  ```powershell
  > npm set init.author.email "wombat@npmjs.com"
  > npm set init.author.name "ag_dubs"
  > npm set init.license "MIT"
  ```

  ### 注意：

  如果 package.json 中没有描述字段，那么 npm 使用 [README.md](http://readme.md/) 或 README 的第一行代替。 该描述可帮助人们在搜索 npm 时找到您的软件包，因此在 package.json 中进行自定义描述以使您的软件包更容易找到是非常有用的。

  ### 如何自定义 package.json 问卷

  如果您希望创建许多 package.json 文件，您可能希望自定义 init 进程期间提出的问题，以便文件始终包含您期望的关键信息。 您可以自定义字段以及问题。

  为此，您需要在主目录 `〜/ .npm-init.js`中创建一个自定义的 **.npm-init.js** 文件。

  一个简单的 **.npm-init.js** 可能看起来像这样：

  ```powershell
  module.exports = {
    customField: 'Custom Field',
    otherCustomField: 'This field is really cool'
  }
  ```

  在主目录中使用此文件运行 `npm init` 会输出一个包含以下行的 package.json 文件：

  ```powershell
  {
    customField: 'Custom Field',
    otherCustomField: 'This field is really cool'
  }
  ```

  您还可以使用提示功能自定义问题。

  ```powershell
    module.exports = prompt("what's your favorite flavor of ice cream, buddy?", "I LIKE THEM ALL");
  ```

  要了解更多关于如何创建高级自定义的信息，请查看 [init-package-json](https://github.com/npm/init-package-json) 的文档。

  ### 指定依赖项

  要指定项目所依赖的包，需要列出要在package.json文件中使用的包。 有两种类型的包可以列出：

  - “dependencies”：这些包是您的应用程序在生产中所需的。
  - “devDependencies”：这些软件包仅用于开发和测试。

  #### 手动编辑 package.json 文件

  你可以手动编辑你的 package.json 。 您需要在包对象中创建一个名为 **dependencies** 的属性，该属性指向一个对象。 该对象将保存用于指定要使用的包的属性。 它将指向一个 [semver](https://docs.npmjs.com/getting-started/docs.npmjs.com/getting-started/semantic-versioning) 表达式，该表达式指定该项目与您的项目兼容的版本。

  如果你有依赖关系，你只需要在本地开发过程中使用，请按照上述相同的说明进行操作，但使用名为 `devDependencies` 的属性。

  例如，下面的项目使用与生产中的 主版本1 匹配的任何版本的包 **my_dep**，并且需要与主版本3 匹配的包 **my_test_framework** 的任何版本，但仅用于开发：

  ```powershell
  {
    "name": "my_package",
    "version": "1.0.0",
    "dependencies": {
      "my_dep": "^1.0.0"
    },
    "devDependencies" : {
      "my_test_framework": "^3.1.0"
    }
  }
  ```

  - **指定版本**：比如`1.2.2`，遵循“大版本.次要版本.小版本”的格式规定，安装时只安装指定版本。
  - **波浪号（tilde）+指定版本**：比如`~1.2.2`，表示安装1.2.x的最新版本（不低于1.2.2），但是不安装1.3.x，也就是说安装时不改变大版本号和次要版本号。
  - **插入号（caret）+指定版本**：比如ˆ1.2.2，表示安装1.x.x的最新版本（不低于1.2.2），但是不安装2.x.x，也就是说安装时不改变大版本号。需要注意的是，如果大版本号为0，则插入号的行为与波浪号相同，这是因为此时处于开发阶段，即使是次要版本号变动，也可能带来程序的不兼容。
  - **latest**：安装最新版本。

  ### --save 和 --save-dev 安装标志

  向 package.json 添加依赖关系的更简单的方法是从命令行执行此操作，使用 --save 或 --save-dev 标记 npm install 命令，具体取决于您希望如何 使用该依赖关系。

  要添加一个条目到你的 package.json 的 **dependencies** 中：

  ```
  npm install <package_name> --save
  ```

  要添加一个条目到你的 package.json 的 **devDependencies** 中：

  ```powershell
  npm install <package_name> --save-dev
  #等效的命令
npm install <package_name> -D
  ```

  ### 管理依赖版本

  npm 使用语义版本控制，或者像我们经常提到的那样使用 SemVer 来管理软件包版本和范围。

  如果您的目录中有一个 package.json 文件，并且运行 **npm install**，那么 npm 将使用语义版本控制来查看该文件中列出的依赖关系并下载最新版本。
  
  ```
  # 只会安装生产环境下的包
npm install --production
  ```
  
  