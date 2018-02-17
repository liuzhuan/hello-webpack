## Plugins 插件

插件是 webpack 的核心内容，它自身的插件系统与配置文件的插件系统一模一样。

与 Loaders 作用于单个文件不同，`plugins` 会对最终的文件或 bundle 进行处理。

只需引入插件并加入到 `plugins` 数组，就可以使用它们了。大部分 `plugins` 可以通过参数配置。由于可以在一个配置文件中多次使用，所以需要 `new` 运算符产生实例。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require('webpack')
const path = require('path')

const config = {
  entry: './app.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' })
  ]
}

module.exports = config
```

### 解析

webpack 插件是一个 JavaScript 对象，包含 `apply` 属性。插件的 `apply` 属性会被 webpack 编译器调用，可以访问整个编译生命周期。

比如：

```js
// ConsoleLogOnBuildWebpackPlugin.js
function ConsoleLogOnBuildWebpackPlugin() {}

ConsoleLogOnBuildWebpackPlugin.prototype.apply = function(compiler) {
  compiler.plugin('run', function(compiler, callback) {
    console.log('The webpack build process is starting!!!')

    callback()
  })
}
```

聪明的你可能已经想到，每个函数同样具有 `apply` 方法。因此，在 webpack 世界中，任何一个函数都可以当作一个插件，可以通过这种方式在配置文件中内联自定义插件。举个例子：

```js
plugins: [
  function() { console.log('anonymous compiler:', this) }
]
```

### Node API

```js
const webpack = require('webpack')
const configuration = require('./webpack.config.js')

let compiler = webpack(configuration)
compiler.apply(new webpack.ProgressPlugin())

compiler.run(function(err, stats) {
  // ...
})
```

上面的例子和 webpack 自身源代码高度相似。webpack 源代码蕴藏着很多宝藏，要善于挖掘利用哦。

### 参考文献

- [Plugins](https://webpack.js.org/concepts/plugins/) - webpack.js.org
- [how to write a plugin](https://github.com/webpack/docs/wiki/how-to-write-a-plugin) - github.com