# Targets

由于 JavaScript 可以在服务器端和浏览器等不同平台中运行，webpack 提供了多个构建目标，可用在 configuration 配置中。

> 注意，不要混淆 `target` 和 `output.libraryTarget` 的用法。

## 用法

只需在 webpack 配置文件中设置 `target` 属性即可：

```js
module.exports = {
  target: 'node'
}
```

按照如上样例，webpack 会按照 Node.js 的平台选项编译。（使用 Node.js 的 `require` 加载代码，不修改 `fs` 或 `path` 等内建模块）

每个目标有各自特定的部署/环境附加项，常见的目标值有：

- `web` 浏览器（默认值）
- `webworker`
- `node`
- `node-webkit`
- `async-node`
- `electron-main`
- `electron-renderer`

## 多目标

尽管 webpack 不支持向 `target` 传入多个字符串，你可以把两个不同配置文件合并，以便输出同构的库：

```js
var path = require('path')
var serverConfig = {
  target: 'node',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.node.js'
  }
}

var clientConfig = {
  target: 'web',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'lib.js'
  }
}

module.exports = [serverConfig, clientConfig]
```

以上的例子会在 `dist` 目录创建 `lib.js` 和 `lib.node.js` 两个文件。

## 资源

具体的不同，可以查看 **compare-webpack-target-bundles** 这个库，它包含了所有的 webpack 目标。

## 参考文献

- [Targets](https://webpack.js.org/concepts/targets/) - webpack.js.org
- [Configuration Target](https://webpack.js.org/configuration/target/) - webpack.js.org
- [compare-webpack-target-bundles | github](https://github.com/TheLarkInn/compare-webpack-target-bundles) - Sean Larkin