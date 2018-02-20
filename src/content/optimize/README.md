# webpack 优化技巧

Google 开发者社区有一篇文章 [Web Performance Optimization with webpack][google] 列举了一些 webpack 优化技巧，摘取其中要点如下。

## 减小前端尺寸

优化的第一步是减小文件体积。

### 开启压缩

webpack 支持两种压缩代码的方法：UglifyJS 插件和 loader 的特殊选项，应同时启用两者。

UglifyJS 插件会在编译后对整个 bundle 压缩。webpack 自带该插件，只需将其加入 `plugins` 数组即可：

```js
/** webpack.config.js */
const webpack = require('webpack')

module.exports = {
  plugins: [
    new webpack.optimize.UglifyJsPlugin()
  ]
}
```

loader 相关的压缩选项，可以压缩 UglifyJsPlugin 无能为力的代码。比如，如果使用 `css-loader` 引入 CSS 文件，样式文件会被编译为字符串。举个例子：

```css
/** comments.css */
.comment {
  color: black;
}
```

将变为：

```js
// minified bundle.js (part of)
exports=module.exports=__webpack_require__(1)(),
exports.push([module.i,".comment {\r\n  color: black;\r\n}",""]);
```

由于 UglifyJs 无法压缩字符串，此时，可设定 loader 的压缩参数：

```js
/** webpack.config.js */
module.exports = {
  module: {
    rule: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          { loader: 'css-loader', options: { minimize: true } }
        ]
      }
    ]
  }
}
```

> 注意，UglifyJS 插件无法压缩 ES2015+（ES6+）代码。这意味着，如果你的代码使用了箭头函数，class 类声明等新语法，并且没有转译为 ES5，插件会报错。

> 如果你需要编译新语法，可以考虑 [uglifyjs-webpack-plugin][uglifyjs] 插件。它和 webpack 自备的 uglifyJS 插件一样，只是更新而已，可以编译 ES2015+ 代码。

### 设定 `NODE_ENV=production`

另一个减小体积的方式是设定环境变量 `NODE_ENV=production`。

第三方库会根据 `NODE_ENV` 的值判断当前的活动环境，是开发环境还是生产环境。有的库会根据 `NODE_ENV` 作不同调整。比如，如果 `NODE_ENV` 不是 `production`，Vue.js 会做额外的判断，还会打印警告消息：

```js
// vue/dist/vue.runtime.esm.js
// ...
if (process.env.NODE_ENV !== 'production') {
  warn('props must be strings when using array syntax.');
}
// …
```

React 也是类似的，在开发环境会增加很多功能。

这些检测和警告信息通常在生产环境是多余的，但是它们只要存在代码中，就会增加体积。可以使用 [DefinePlugin][define-plugin] 移除这些多余代码：

```js
/** webpack.config.js */
const webpack = require('webpack')

module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"production"',
    }),
    new webpack.optimize.UglifyJsPlugin()
  ],
}
```

`DefinePlugin` 会将所有特定变量出现的地方替换为指定的数值。然后，`UglifyJS` 插件会删除这些无效分枝，因为它知道这些逻辑分枝永远为假。

> 注意，并不一定必须使用 UglifyJsPlugin。任何支持 dead code removal 的工具都可以，比如 [Babel Minify plugin][babel-minify] 或者 [Google Closure Compiler plugin][webpack-closure]。

### 使用 ES 模块

前端减肥的下一步是使用 [ES 模块][es-module]。

如果使用 ES 模块，webpack 可以执行 `tree-shaking`。Tree-shaking 就是删除 ES 模块无用的代码。

[（未完待续）](https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size#use_es_modules)

## 使用长期缓存

## 监测分析应用

## 总结

## REF

- [Web Performance Optimization with Webpack - Google Developers][google]
  - [Decrease Front-end Size][step1], by Ivan Akulov
- [webpack training project][training]
- [uglifyjs-webpack-plugin - github][uglifyjs]

[google]: https://developers.google.com/web/fundamentals/performance/webpack/
[step1]: https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size
[training]: https://github.com/GoogleChromeLabs/webpack-training-project
[bundle-buddy]: http://www.susielu.com/data-viz/bundle-buddy
[uglifyjs]: https://github.com/webpack-contrib/uglifyjs-webpack-plugin
[define-plugin]: https://webpack.js.org/plugins/define-plugin/
[webpack-closure]: https://github.com/roman01la/webpack-closure-compiler
[babel-minify]: https://github.com/webpack-contrib/babel-minify-webpack-plugin
[es-module]: https://ponyfoo.com/articles/es6-modules-in-depth