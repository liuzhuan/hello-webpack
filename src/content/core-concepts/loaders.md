## Loaders

webpack 将所有类型的文件（包括 .css, .html, .scss, .jpg 等）都当作模块。但是 webpack 只懂 JavaScript 。因此，加入到依赖图前，所有文件会被 Loaders 统一转换为 JavaScript 模块。

Loaders 对模块的源文件进行处理。类似于其他构建工具的 `task`。

配置文件中的 loaders 大体有两个目的：

1. 指明哪些文件需要被处理（通过 `test` 属性）
2. 如何处理这些文件，以便加入到依赖图，打包到 bundle 。（`use` 属性）

```js
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
  }
}

module.exports = config
```

上面的配置文件中，`module.rules` 是一个数组，其中的每个元素至少包含 `test` 和 `use` 两个必填属性。

> 注意，规则定义在 `module.rules` 中，而不是 `rules` 中。

### 例子

如果你想让 webpack 加载 css 文件，或把 TypeScript 转换为 JavaScript，需要首先安装用到的 loaders:

```
npm install --save-dev css-loader
npm install --save-dev ts-loader
```

然后配置 webpack :

```js
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
}
```

### Loader 特性

- 可链式调用。它们通过管道对原始资源施加操作。loader 数组的执行顺序是和在数组的索引位置反向的。第一个 loader 的输出值会作为下一个的输入值，在最后的 loader，应该返回一个 JS 文件。
- Loaders 可以输出附加的任意文件。

### 解析 Loaders

Loader 遵循标准的模块解析规则。大多数情况下，它来自于模块路径（比如 `npm install`，`node_modules` 之类的）。

loader 模块需要暴露一个函数，使用 Node.js 兼容语法（也就是不要用 ES6 模块写法）。通常它们都是由 npm 管理，但你也可以在应用程序中使用自定义 loaders。

按照惯例，loader 的命名类似于 `xxx-loader`（比如 `json-loader`）。

### 参考文献

- [Loaders](https://webpack.js.org/concepts/loaders/) - webpack.js.org