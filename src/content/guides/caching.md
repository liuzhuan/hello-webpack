## 缓存 Caching

正确使用缓存可有效减少 HTTP 请求次数，节约带宽和下载时间。

### Output Filenames

使用 `[chunkhash]` 可以在文件名后增加与内容相关的哈希值。

> `[hash]` 生成的哈希值与构建相关，此处不如 `[chunkhash]` 有效。

**webpack.config.js**

```js
output: {
  filename: '[name].[chunkhash].js',
}
```

运行构建脚本 `npm run build`，会输出打包后的脚本，脚本名称的哈希值反映了打包文件的内容。

如果不修改文件内容，再次运行打包，会发现哈希值有变动。为什么？

这是因为 webpack 在入口文件包含了一些模版代码，尤其是运行时和 manifest 清单。

### 提取模版代码

人们都知道 `CommonsChunkPlugin` 能把模块分离为不同的 bundle。但是它还有一个不那么出名的功能：把每次构建都变化的 webpack 模板代码提取出来。通过指定一个不在 `entry` 配置出现的名称，它就会把模板代码自动提取出，放置到一个独立文件内。

**webpack.config.js**

```js
const webpack = require('webpack')

module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime'
    })
  ]
}
```

当再次构建后，会发现新增的 `runtime.xxx.js` 文件。

根据最佳实践，一般将第三方库文件，比如 `lodash` 或 `react` 打包到独立的 `vendor` 包，因为它们相对稳定，不易变化。这个步骤可以让客户端请求更少的文件。这可以通过在 entry 新增入口文件，同时在 plugins 中新增 `CommonsChunkPlugin` 实例来实现：

**webpack.config.js**

```js
module.exports = {
  entry: {
    main: './src/index.js',
    vendor: [
      'lodash'
    ]
  },

  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime'
    }),
  ]
}
```

注意，**插件的顺序很重要，`CommonsChunkPlugin` 的 `vendor` 实例必须出现在 `runtime` 实例之前**。

再次构建后，将喜获 `vendor` 包。

### 模块 ID

如果新增 `print.js` 自定义模块，我们预期只有 `main` 包的哈希值会变化，但实际上 `vendor`, `main` 和 `runtime` 三者的哈希值同时发生了变化。

这是因为根据默认的解析顺序，每个模块的 `module.id` 发生了变化。所以，小结一下：

- `main` 包是因为新增了内容
- `vendor` 是因为 `module.id` 改变
- `runtime` 是因为包含新增模块的引用

第 1 个和第 3 个都在预料之中，如何修复第 2 个 `vendor` 的变化呢？

幸运的是，有两个插件可以解决这个问题。第一个插件是 `NamedModulesPlugin` 会使用模块路径来代替数字标识符。尽管这个插件在开发环境可以生成更可读的输出信息，但它运行速度确实慢。另一个插件是 `HashedModuleIdsPlugin`，推荐在生产环境中使用：

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new webpack.HashedModuleIdsPlugin(),
    new webpack.optimize.CommonsChunkPlugin({name: 'vendor'}),
    new webpack.optimize.CommonsChunkPlugin({name: 'runtime'}),
  ]
}
```

现在，不管本地依赖如何变化，`vendor` 包的哈希值应该始终保持一致。

### REF

- https://webpack.js.org/guides/caching/