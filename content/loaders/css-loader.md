## css-loader

### 安装

```
npm install --save-dev css-loader
```

### 用法 Usage

`css-loader` 将 css 文件中的 `@import` 和 `url()` 解释为 js 模块的 `import/require()` ，从而可以构建样式的“依赖图”。

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  }
}
```

你还可以使用 css-loader 导出字符串，就像在 Angular 的组件样式：

**webpack.config.js**

```js
{
  test: /\.css$/,
  use: [
    'to-string-loader',
    'css-loader'
  ]
}
```

### REF

- https://webpack.js.org/loaders/css-loader/
- https://github.com/webpack-contrib/css-loader