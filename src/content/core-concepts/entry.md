## Entry

在 webpack 中，应用程序的入口称为 *entry point*，通过配置文件的 `entry` 属性定义。

### 单入口语法

用法：`entry: string | Array<string>`

```js
const config = {
  entry: './app.js'
}

module.exports = config
```

以上的代码，其实是下面的快捷方式：

```js
const config = {
  entry: {
    main: './app.js'
  }
}
```

> 当你向 `entry` 中传入数组时发生了什么？这会产生所谓的“multi-main entry”。这通常用于将多个相互依赖的文件合并到一起，并把它们的依赖关系放入一个 `chunk`。

单入口语法简单，但扩展性差。

### Object 语法

用法：`entry: {[entryChunkName: string]: string | Array<string>}`

```js
const config = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
}
```

这种方式稍显复杂，但扩展性极佳。

> "可扩展 webpack 配置" 指的是可复用和可组合的配置文件。在多个不同环境，不同构建目标和运行时的构建中常常会使用这种技巧。可以使用 `webpack-merge` 等专用工具将多个不同配置参数合并。

### 参考文献

- [Entry Points](https://webpack.js.org/concepts/entry-points/) - webpack.js.org