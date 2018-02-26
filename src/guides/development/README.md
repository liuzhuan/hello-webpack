# 开发

> 本节的代码例子基于[输出管理][output]的指南。本节的工具**只适合开发阶段**，**一定要避免在生产线上使用**！

## 使用 source maps

当 webpack 打包源代码后，从错误警告中很难找到问题的原始位置。比如，你将三个源文件（`a.js`, `b.js` 和 `c.js`）打包成一个（`bundle.js`），其中一个包含错误，堆栈跟踪（`stack trace`）只会简单的指向 `bundle.js`。这通常没什么意义，因为你要知道错误到底来自哪个文件。

JavaScript 提供了 [source maps][source-maps] ，将编译后代码映射到源文件，方便开发者追踪错误和警告。如果错误源自 `b.js`，source map 会精准定位到原始源文件的错误位置。

source map 有许多不同的[配置参数][devtool]，一定要熟悉各个参数的作用，才能得到满足需求的配置。

在本指南中，我们使用 `inline-source-map` 选项，便于展示（但并不适合生产环境）：

```js
/** webpack.config.js */
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackPlugin = require('clean-webpack-plugin')

module.exports = {
  entry: {
    app: './src/index.js',
    print: './src/print.js'
  },
  devtool: 'inline-source-map',
  plugins: [
    new CleanWebpackPlugin(['dist']),
    new HtmlWebpackPlugin({
      title: 'Development'
    })
  ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

## 选择一个开发工具

> 一些文本编辑器拥有“安全写”的功能，可能会和下面的某些工具冲突。阅读[调整你的文本编辑器][adjust-editor]，找到合适的解决方案。

手动输入 `npm run build` 效率太低。在 webpack 中有一些解决方案帮助你自动编译代码：

1. webpack 的 Watch 模式
2. webpack-dev-server
3. webpack-dev-middleware

大多数情况下，你可能只需要 `webpack-dev-server` 即可，我们也来认识一下所有的选项。

### 使用 Watch 模式

你可以指定 webpack 监视所有的依赖图的文件，一旦其中文件有更新，就会自动编译代码，因此你就无需手动输入构建命令。

让我们增加一个 npm 脚本，开启 webpack 的 Watch 模式：

```json
{
  "scripts": {
    "watch": "webpack --watch"
  }
}
```

从此，可以在命令行输入 `npm run watch`，就会看到 webpack 自动编译代码。

这种方式的一个缺点是你需要手动刷新浏览器，才能看到变化。如果能自动刷新浏览器就更好了，而这，正是 `webpack-dev-server` 的强项。

### 使用 `webpack-dev-server`

`webpack-dev-server` 提供了一个简单浏览器，可以实现自动刷新页面。首先安装它：

```sh
npm install --save-dev webpack-dev-server
```

在配置文件增加对应选项，告知服务器在那里查找文件：

```js
/** webpack.config.js */
module.exports = {
  // ...
  devServer: {
    contentBase: './dist'
  },
  // ...
}
```

以上选项告知 `webpack-dev-server` 将 `localhost:8080` 映射到 `dist` 目录下的文件。

我们可以增加一个全新 npm 脚本：

```json
/** package.json */
{
  "scripts": {
    "start": "webpack-dev-server --open"
  }
}
```

现在，我们在终端运行 `npm start`，就会看到浏览器自动打开，并加载我们的页面。如果修改了任何源文件并保存，服务器就会在代码编译后自动重载。试一试吧！

`webpack-dev-server` 有超多选项，可以去[相关文档][dev-server-config]了解更多细节。

> 既然服务器跑起来了，何不试试[模块热更新][hmr]！

### 使用 webpack-dev-middleware

`webpack-dev-middleware` 是一个封装器，可以把 webpack 处理的文件发送到服务器。`webpack-dev-server` 内部使用的就是它。它还可以作为一个独立的包存在，从而允许更深度的定制。我们可以看看如何结合 `webpack-dev-middleware` 和 express 服务器做开发。

首先安装 `express` 和 `webpack-dev-middleware`：

```sh
npm install --save-dev express webpack-dev-middleware
```

现在我们需要对 wepback 配置文件做一些调整，以便让中间件正常运行：

```js
/** webpack.config.js */
module.exports = {
  output: {
    // ...
    publicPath: '/'
  }
}
```

`publicPath` 也会用在服务器脚本中，以便确保文件可以在 `http://localhost:300` 正常访问。下一步就是设置我们自己的 `express` 服务器，文件目录结构如下：

```
webpack-demo
|- package.json
|- webpack.config.js
|- server.js
|- /dist
|- /src
  |- index.js
  |- print.js
|- /node_modules
```

```js
/** server.js */
const express = require('express')
const webpack = require('webpack')
const webpackDevMiddleware = require('webpack-dev-middleware')

const app = express()
const config = require('./webpack.config.js')
const compiler = webpack(config)

/**
* 告知 express 使用 webpack-dev-middleware，
* 同时将 webpack.config.js 作为基础配置
*/
app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publishPath
}))

app.listen(3000, function(){
  console.log('Example app listening on port 3000!\n')
})
```

现在添加一个 npm 脚本：

```json
/** package.json */
{
  "scripts": {
    "server": "node server.js"
  }
}
```

现在，在终端输入 `npm run server`，打开浏览器，输入地址 `http://localhost:3000`，应该就能看到 webpack app 的运行了！

> 如果想了解更多模块热更新的工作原理，推荐你阅读[模块热更新指南][hmr]。

## 调整你的文本编辑器

当使用你的自动编译功能时，保存文件时你可能会遇到一些问题。一些编辑器有“安全写入”特性，有可能会影响重新编译。

为了禁用这些功能，方法如下：

- **Sublime Text 3** - 在用户偏好设置中增加 `atomic_save: "false"`
- **IntelliJ** - 在偏好中搜索 `safe write` ，关闭它
- **Vim** - 在设置中增加 `:set backupcopy=yes`
- **WebStorm** - 在 `偏好设置 > 外观&行为 > 系统设置` 取消选中 `"safe write"`

（完）

## REF

- [Development - webpack.js.org][guides]
- [Output Management - webpack.js.org](output)
- [An Introduction to Source Maps][source-maps], by *Matt West*

[guides]: https://webpack.js.org/guides/development/
[output]: https://webpack.js.org/guides/output-management
[source-maps]: http://blog.teamtreehouse.com/introduction-source-maps
[devtool]: https://webpack.js.org/configuration/devtool/
[adjust-editor]: https://webpack.js.org/guides/development/#adjusting-your-text-editor
[dev-server-config]: https://webpack.js.org/configuration/dev-server
[hmr]: https://webpack.js.org/guides/hot-module-replacement/