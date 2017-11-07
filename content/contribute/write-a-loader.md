## 编写 Loader

loader 是一个 node 模块，可以导出一个函数。当资源需要被 loader 转换时，就会调用这个函数。这个函数可以通过 `this` 上下文调用 Loader API

### 配置 Setup

下面介绍一下如何在本地开发和测试 loader

为了测试单一 loader，只需在 rule 对象中使用 `path.resolve` 解析 loader 路径即可：

**webpack.config.js**

```js
{
  test: /\.js$/,
  use: [
    {
      loader: path.resolve('path/to/loader.js'),
      options: { /* ... */ }
    }
  ]
}
```

为了测试多个 loader，可以利用 `resolveLoader.modules` 选项，指示 webpack 去哪里搜寻 loaders。比如，如果项目中你有一个本地的 `/loaders` 目录：

**webpack.config.js**

```js
resolveLoader: {
  modules: [
    'node_modules',
    path.resolve(__dirname, 'loaders')
  ]
}
```

另外，如果你有一个独立的 loader 仓库，可以通过 `npm link` 映射到当前项目，以便测试。

### 简单应用

当一个 loader 应用到资源上时，loader 的参数只有一个 - 包含资源原始内容的字符串。

同步 loader 只需 `return` 代表转换模块的单一数据。在更复杂的场景下，loader 可以返回任意数据，这个需要使用 `this.callback(err, values...)` 函数。错误要么传入 `this.callback` 要么在同步 loader 中抛出。

> 未完待续...

### REF

- https://webpack.js.org/contribute/writing-a-loader/
- https://webpack.js.org/api/loaders/
- https://docs.npmjs.com/cli/link
- https://webpack.github.io/docs/how-to-write-a-loader.html 