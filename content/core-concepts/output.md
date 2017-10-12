## Output

配置文件的 `output` 参数告诉 webpack 如何处理打包的代码。

```js
const path = require('path')

module.exports = {
  entry: './app.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  }
}
```

`output` 有很多可配置的参数。