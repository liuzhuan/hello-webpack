# 模块热更新

模块热更新（`Hot Module Replacement`, HMR）是 webpack 提供的一个强大特性。它允许各种模块在运行时更新，而不用整体刷新页面。本页面主要关注具体做法，[概念页][concepts]提供了更多工作原理细节。

> ✨ 注意：HMR 不适合生产环境，意味着它只能在开发环境中启用。查阅[线上构建指南][production]了解更多信息。

## 开启 HMR

这个特性可以极大提高开发效率。我们只需要更新 `webpack-dev-server` 配置，使用 webpack 内置的 HMR 插件即可。我们还需要移除 `print.js` 的入口文件，因为它会被 `index.js` 模块消费。

> 如果你使用了 `webpack-dev-middleware` 而不是 `webpack-dev-server`，请使用 [`webpack-hot-middleware`][webpack-hot-middleware] 在你的服务器开启 HMR。

```js
/** webpack.config.js */
// ...
const webpack = require('webpack')

module.exports = {
  entry: {
    app: './src/index.js'
  },
  devServer: {
    contentBase: './dist',
    hot: true
  },
  plugins: [
    // ...
    new webpack.NamedModulesPlugin(),
    new webpack.HotModuleReplacementPlugin()
  ]
}
```

> 还可以使用命令行修改 `webpack-dev-server` 的配置项，比如：`webpack-dev-server --hotOnly`

注意，我们同时增加了 `NamedModulesPlugin` 来更清楚看到哪个依赖项被打补丁了。接着，我们使用 `npm start` 启动开发服务器。

我们现在来更新 `index.js` 文件，当监测到 `print.js` 有修改时，我们告知 webpack 来接收这些改变。

```js
/** index.js */
import _ from 'lodash'
import printMe from './print.js'

function component() {
  var element = document.createElement('div')
  var btn = document.createElement('button')

  element.innerHTML = _.join(['Hello', 'webpack'], ' ')

  btn.innerHTML = 'Click me and check the console!'
  btn.onclick = printMe

  element.appendChild(btn)

  return element
}

document.body.appendChild(component())

if (module.hot) {
  module.hot.accept('./print.js', function() {
    console.log('Accepting the updated printMe module!')
    printMe()
  })
}
```

## REF

- [Hot Module Replacement][hmr] - webpack.js.org

[hmr]: https://webpack.js.org/guides/hot-module-replacement/
[concepts]: ../../concepts/hmr/README.md
[production]: https://webpack.js.org/guides/production
[webpack-hot-middleware]: https://github.com/glenjamin/webpack-hot-middleware