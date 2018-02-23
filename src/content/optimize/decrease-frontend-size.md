# 减小前端体积

优化的第一步是减小文件体积。

## 开启压缩

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

loader 相关的压缩选项，可以压缩 `UglifyJsPlugin` 无能为力的代码。比如，如果使用 `css-loader` 引入 CSS 文件，样式文件会被编译为字符串。举个例子：

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

由于 UglifyJs 无法压缩字符串，此时，可设定 `css-loader` 的压缩参数 `minimize`：

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

## 设定 `NODE_ENV=production`

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

这些检测和警告信息通常在生产环境是多余的，只要存在代码中，就会增加体积。可以使用 [DefinePlugin][define-plugin] 移除这些多余代码：

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

`DefinePlugin` 会将所有指定的变量替换为对应的数值。接着，`UglifyJS` 插件会删除这些无效的逻辑分支，因为它知道这些逻辑分支的判断条件永远为假。

> 注意，并不一定必须使用 UglifyJsPlugin。任何支持 dead code removal 的工具都可以，比如 [Babel Minify plugin][babel-minify] 或者 [Google Closure Compiler plugin][webpack-closure]。

## 使用 ES 模块

前端减肥的下一步是使用 [ES 模块][es-module]。

如果使用 ES 模块，webpack 可以执行 `tree-shaking`。Tree-shaking 可以删除 ES 模块中用不到的代码。

比如，我们有下面的代码：

```js
/** foo.js */
export const render = () => { return 'Rendered' }
export const commentRestEndpoint = '/rest/comments'

/** app.js */
import { render } from './foo'
render()
```

webpack 知道，`foo.js` 中，`commentRestEndPoint` 代码无需输出，因此生成如下 bundle：

```js
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict";
  const render = () => { return 'Rendered' }
  /* harmony export (immutable) */ __webpack_exports__["a"] = render;
  const commentRestEndpoint = '/rest/comments'
  /* unused harmony export commentRestEndpoint */
})
```

接下来，`UglifyJsPlugin` 会移除无效代码。最终结果如下

```js
function(e,t,r){"use strict";t.a=(()=>"Rendered")
```

> 使用 commonJS 模块，webpack 不会开启 tree-shaking 功能，所有代码均会打包到最终 bundle。

注意，在 webpack 中，必须使用压缩器才能实现 tree-shaking。webpack 只是将未使用的代码不做导出处理，真正移除无效代码的是 **UglifyJsPlugin**。因此，如果没有使用压缩器，代码体积并不会减小。

> ⚠️ 警告：千万不要把 ES 模块编译为 CommonJS 模块。

> 如果使用 Babel 转译器，并且开启了 `babel-preset-env` 或 `babel-preset-es2015` 预设值，一定要仔细检查这些配置。它们默认会把 ES 的 `import` 和 `export` 编译为 CommonJS 的 `require` 和 `module.exports`。设定 `{ modules: false }` 可以禁止该默认行为。

> TypeScript 也是一样，记得要在 `tsconfig.json` 中设定 `{ "compilerOptions": { "module": "es2015" } }` 。

延伸阅读
- [ES6 Modules in Depth](https://ponyfoo.com/articles/es6-modules-in-depth), by Nicolás Bevacqua, 2015/09/25
- [webpack 文档： Tree Shaking](https://webpack.js.org/guides/tree-shaking/)

## 优化图像

图像占据了[页面体积的一半][stats]以上。尽管它们不如 JavaScript 那么重要（比如，它们不会阻塞渲染），但依然消耗着大部分带宽。在 webpack 中可以使用 `url-loader`, `svg-url-loader` 和 `image-webpack-loader` 优化图像。

`url-loader` 会把小型静态资源内联到页面中。没有配置情况下，它会把原始图像拷贝到编译的 bundle 同级目录，并返回该资源的 url 地址。如果设置了 `limit` 选项，它会把小于该限制的资源编译为 [Base64 data url][data-uris]，并返回该 url。这会把图像内联到 JavaScript 中，从而减少一个 HTTP 请求：

```js
/** webpack.config.js */
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif)$/,
        loader: 'url-loader',
        options: {
          // 内联小于 10 kB 的图像（10240 bytes）
          limit: 10 * 1024
        }
      }
    ]
  }
}
```

```js
/** index.js */
import imageUrl from './image.png'

/**
* 如果 image.png 小于 10 kB，imageUrl 会包含编译后的代码。比如：`data:image/png;base64,ivbor2sdo...`
* 否则，image.png 会包含它的 url 地址，比如：`/2fcd56a1920.png`
*/
```

> ✨ 注意：内联图像会降低请求数量，这确实是好事。但会增加下载和解析时间，并且会增大内存消耗。务必不要内联大尺寸图像，也要控制内联图像的总量，否则增加的 bundle 时间会抵消内联的优势。

`svg-url-loader` 和 `url-loader` 工作原理相似，只不过它使用 [URL 编码][url-enc]，而不是 Base64 编码。这对 SVG 图像很有用，因为 SVG 就是普通文本，这个编码体积更小：

```js
/** webpack.config.js */
module.exports = {
  module: {
    rules: [
      {
        test: /\.svg$/,
        loader: 'svg-url-loader',
        options: {
          // 内联小于 10 kB 的图像（10240 比特）
          limit: 10 * 1024,
          // 删除 url 的引号（因为大部分情况下是多余的）
          noquotes: true
        }
      }
    ]
  }
}
```

> ✨ 注意：`svg-url-loader` 有些选项可以增强 IE 支持度，但是会对其他浏览器的内联造成坏的影响。如果需要支持 IE，可以设置 `iesafe: true` 选项。

`image-webpack-loader` 压缩图像，支持 JPG, PNG, GIF 和 SVG，所以这些类型都可以使用。

该 loader 无法将图像内嵌到页面，因此必须和 `url-loader` 和 `svg-url-loader` 配合使用。为了避免在多个 rules 中复制粘贴（一个针对 JPG/PNG/GIF 图像，另一个针对 SVG），我们可以包含一个单独的 rule，并设置 [`enforce: 'pre'`][rule-enforce] 选项：

```js
/** webpack.config.js */
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif|svg)$/,
        loader: 'image-webpack-loader',
        /** 该属性会让此 loader 优先执行 */
        enforce: 'pre'
      }
    ]
  }
}
```

loader 的默认配置已经足够好了。如果你想更进一步配置，可以查看[插件选项][image-webpack-loader]。如果不知道如何设置，可以查看 Addy Osmani 的[有关图像优化的建议][images-guide]。

延伸阅读

- [base64 的用途是什么？](https://stackoverflow.com/questions/201479/what-is-base-64-encoding-used-for) - stackoverflow
- Addy Osmani 的[图片优化建议](https://images.guide) 👍

## 优化依赖

平均超过一半的 JavaScript 体积来自依赖，而且其中一部分可能还是多余的。

比如，Lodash（v4.17.4）会对最终的打包文件贡献 72 KB 的代码量。如果你仅使用了其中 20 个函数，那么剩下的 65 KB 代码就是多余的。

另一个例子是 Momenet.js。它的 2.19.1 版本压缩后占据 223 KB，的确很大 - 据统计，2017年10月的 JavaScript 平均体积是 452 KB。但是 Moment.js 中 170 KB 代码都是本地化相关的，如果你不需要在 Moment.js 中使用多语种，这些多出来的 170 KB 就毫无意义。

这些多余的依赖可被轻松优化。我们在 Github 仓库中搜集了优化方法，[看这里][webpack-libs-optimize]！

## 开启 ES 模块拼接（即作用域提升 scope hoisting）

当你构建 bundle 时，webpack 会把每个模块包裹成一个函数：

```js
/** index.js */
import { render } from './comments.js'
render()

/** comments.js */
export function render(data, target) {
  console.log('Rendered!')
}
```

会转变为：

```js
/** bundle.js (part of) */
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict"
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true })
  var __WEBPACK_IMPORTED_MODULE_0__comments_js__ = __webpack_require__(1)
  Object(__WEBPACK_IMPORTED_MODULE_0__comments_js__["a" /* render */])()
}),
/* 1 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict"
  __webpack_exports__["a"] = render
  function render(data, target) {
    console.log('Rendered!')
  }
})
```

过去，为了隔离 CommonJS/AMD 模块，必须这么做。但这种做法会增大每个模块的体积和运行开销。

webpack 2 引入了对 ES 模块的支持。与 CommonJS 和 AMD 模块不同，打包时不再使用函数包裹每个模块。webpack 3 通过 `ModuleConcatenationPlugin` 让打包成为可能。以下是插件的用法：

```js
/** index.js */
import { render } from './comments.js'
render()

/** comments.js */
export function render(data, target) {
  console.log('Rendered!')
}
```

会变为：

```js
/** 与上次的打包不同，本次输出只包含一个模块，它拥有两个文件的内容 */

/** bundle.js (part of; 应用了 ModuleConcatenationPlugin) */
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict"
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true })
  // CONCATENATED MODULE: ./comments.js
  function render(data, target) {
    console.log('Rendered!')
  }

  // CONCATENATED MODULE: ./index.js
  render()
})
```

看到区别了吧？在普通 bundle 中，模块 0 从 模块 1 中引入 `render` 函数。使用 `ModuleConcatenationPlugin` 插件后，`require` 直接替换为目标函数，模块 1 被移除。bundle 的模块数量降低，从而也减少了模块运行消耗。

要开启该行为，只需在 `plugins` 数组中增加 `ModuleConcatenationPlugin` 即可：

```js
/** webpack.config.js */
const webpack = require('webpack')

module.exports = {
  plugins: [
    new webpack.optimize.ModuleConcatenationPlugin()
  ]
}
```

> ✨ 注意：想知道为什么这个特性没有默认开启吗？模块拼接（Concatenating Modules）确实很酷，但[会造成更长的编译时间，还会破坏模块热更新][scope-hoist-issue]。因此，它只能在生产环境开启。

延伸阅读

- [ModuleConcatenationPlugin 的文档][module-concatenation-plugin]
- [作用域提升的简介][breif-scope-hoisting]
- 该插件工作原理的[详细描述][detailed-module-concat-plugin]

## 使用 `externals` 如果 webpack 和非 webpack 代码并存

在一些大型项目中，会出现部分代码使用 webpack 编译，还有部分代码使用其他构建工具的情况。比如，视频托管网站中，播放器组件可能使用 webpack 构建，周围的页面元素可能使用其他构建工具。

如果两者有公共的依赖，可以通过分享代码避免多次下载。这可以通过 [webpack 的 externals][externals] 选项实现，它会将模块替换为变量或者其他外部引入模块。

### 如果依赖存在于 `window`

如果你的非 webpack 代码依赖于 `window` 中的变量，即依赖变量名称：

```js
/** webpack.config.js */
module.exports = {
  externals: {
    'react': 'React',
    'react-dom': 'ReactDOM'
  }
}
```

通过该配置，webpack 将不会打包 `react` 和 `react-dom`。它们会被替换为类似如下代码：

```js
// bundle.js (part of)
(function(module, exports) {
  // A module that exports `window.React`. Without `externals`,
  // this module would include the whole React bundle
  module.exports = React;
}),
(function(module, exports) {
  // A module that exports `window.ReactDOM`. Without `externals`,
  // this module would include the whole ReactDOM bundle
  module.exports = ReactDOM;
})
```

### 如果依赖通过 AMD 模块引入

如果非 webpack 代码没有向 `window` 中暴露依赖，事情就会更加复杂。如果这些依赖是 AMD 模块形式，还是可以避免多次加载同样的代码。

为了实现该目的，可以把 webpack 代码编译为 AMD 模块，并且将依赖的模块别名重定向到库 URL 地址：

```js
/** webpack.config.js */
module.exports = {
  output: {
    libraryTarget: 'amd'
  },

  externals: {
    'react': { amd: '/libraries/react.min.js' },
    'react-dom', { amd: '/libraries/react-dom.js' }
  }
}
```

webpack 会使用 `define()` 打包这些 bundle ，使其依赖这些 URL 地址：

```js
// bundle.js (beginning)
define(["/libraries/react.min.js", "/libraries/react-dom.min.js"], function () { … });
```

如果非 webpack 代码使用同样的 URL 地址加载依赖，那么这些文件就会只加载一次，后续请求会使用缓存。

> ✨ 注意：webpack 仅会替换 `externals` 对象键值对应的导入对象。这意味着，如果你的代码是 `import React from 'react/umd/react.production.min.js'`，这个库不会从 bundle 中排除。这是合理的，因为 webpack 不知道 `import 'react'` 和 `import 'react/umd/react.production.min.js'` 是否是一回事。谨记这一点。

延伸阅读

- webpack [有关 externals][externals] 的官方文档

## 总结

- 使用 `UglifyJsPlugin` 和 loader 选项压缩代码
- 使用 `DefinePlugin` 移除开发环境的代码
- 使用 ES 模块开启 tree shaking
- 压缩图像
- 开启依赖特有的优化
- 开启模块拼接
- 依据个人实际情况，酌情使用 `externals`

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
[stats]: http://httparchive.org/interesting.php
[data-uris]: https://css-tricks.com/data-uris/
[url-enc]: https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding
[rule-enforce]: https://webpack.js.org/configuration/module/#rule-enforce
[image-webpack-loader]: https://github.com/tcoopman/image-webpack-loader#options
[images-guide]: https://images.guide/
[webpack-libs-optimize]: https://github.com/GoogleChromeLabs/webpack-libs-optimizations
[scope-hoist-issue]: https://twitter.com/TheLarkInn/status/925800563144454144
[module-concatenation-plugin]: https://webpack.js.org/plugins/module-concatenation-plugin/
[breif-scope-hoisting]: https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f
[detailed-module-concat-plugin]: https://medium.com/webpack/webpack-freelancing-log-book-week-5-7-4764be3266f5
[externals]: https://webpack.js.org/configuration/externals/