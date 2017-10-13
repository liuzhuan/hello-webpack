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

### 参考文献

- [Loaders](https://webpack.js.org/concepts/loaders/) - webpack.js.org