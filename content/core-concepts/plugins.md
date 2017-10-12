## Plugins

Loaders 只会作用于单个文件，`plugins` 会对最终的文件或包括进行处理。

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