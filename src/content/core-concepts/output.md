## Output

配置文件的 `output` 参数告诉 webpack 如何处理打包的代码。注意，尽管可以有多个入口，但只能有一个 `output`。

### 用法

`output` 参数至少要包含以下两个属性：

- `filename` 指定输出文件的名称
- 绝对路径 `path` 指定输出路径

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

### 多入口

如果你的配置需要输出不只一个 "chunk"，你需要使用*替换语法*保证每个文件都有独特的名称。

```js
{
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
}
```

常用的替换占位符有：

- `[name]`: 入口名称
- `[id]`: 内部 chunk id
- `[hash]`: 每次构建生成的唯一哈希值
- `[chunkhash]`: 基于每个 chunk 内容的哈希值

### 高级用法

下面是一个使用 CDN 和素材哈希的更复杂的案例：

```js
output: {
  path: '/home/proj/cdn/assets/[hash]',
  publicPath: 'http://cdn.example.com/assets/[hash]/'
}
```

如果不知道最终的 `publicPath`，在你的入口文件中设定 `__webpack_public_path__`：

```js
__webpack_public_path__ = myRuntimePublicPath

// rest of your application entry
```

### 参考文献

- [Output](https://webpack.js.org/concepts/output/) - webpack.js.org
- [Output substitution](https://webpack.js.org/configuration/output/#output-filename) - webpack.js.org