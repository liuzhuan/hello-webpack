# 模块

webpack 支持的模块类型比 Node.js 中更多，比如：

- ES2015 模块，使用 `import` 引入
- CommonJS 模块，使用 `require()` 引入
- AMD 模块，使用 `define` 和 `require`
- css/less/sass 模块，使用 `@import` 引入
- 样式表中的图片，使用 `url(...)` 引入
- html 文件中的图片，使用 `<img src=...>` 引入

>  ES2015 模块在 webpack 1 使用时，需要依赖一个 loader ，在 webpack 2 中开箱即用，无需 loader 。

## 参考文献

- [Modules](https://webpack.js.org/concepts/modules/) - webpack.js.org