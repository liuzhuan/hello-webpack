# 开启长期缓存

可以改善应用下载时间的第二步（第一步是[压缩体积][step1]）就是开启缓存。这样应用的部分资源就可以驻留在客户端，防止每次访问都重新下载。

## 使用 bundle 版本号和缓存头

使用缓存的一般步骤是：

1. 通知浏览器对某个资源缓存很长时间（比如，一年）：

```
# 服务器头
Cache-Control: max-age=31536000
```

如果你不熟悉 `Cache-Control` 的用法，可参考 Jake Archibald 的[关于缓存的最佳实践][caching-best]精彩博文。

2. 当文件内容变化时，重命名文件，以便重新下载：

```html
<!-- 改变前 -->
<script src="./index-v15.js"></script>

<!-- 改变后 -->
<script src="./index-v16.js"></script>
```

通过 webpack 可以达到同样目的。但是不是用版本号，而是指定文件哈希值。为了在文件名中包含哈希值，可以使用 `[chunkhash]` ：

```js
/** webpack.config.js */
module.exports = {
  entry: './index.js',
  output: {
    filename: 'bundle.[chunkhash].js'
  }
}
```

> ✨ 注意：即使 bundle 内容一样，webpack 也可能产生不同的哈希值 - 比如，重命名文件或在不同操作系统编译打包。这是[已知 bug][hash-bug]。

> 如果需要把文件名称发送到客户端，可以使用 **HtmlWebpackPlugin** 或 **WebpackManifestPlugin** 。

[`HtmlWebpackPlugin`][html-webpack-plugin] 是一个简单的方案，缺点是不太灵活。在编译过程中，它会产生一个 HTML 文件，该文件包含所有的编译资源。如果你的服务器逻辑不太复杂，这个插件就够用了：

```html
<!-- index.html -->
<!doctype html>
<!-- ... -->
<script src="bundle.8e0d62a03.js"></script>
```

[`WebpackManifestPlugin`][webpack-manifest-plugin] 是一个更灵活的方案，适用于复杂逻辑的服务器。编译过程中，它会产生一个 JSON 文件，其中包含一个映射：没有哈希值的文件名与有哈希值文件名的映射。通过该 JSON 文件就可以知道使用哪个文件：

```json
{
  "bundle.js": "bundle.8e0d62a03.js"
}
```

延伸阅读

- Jake Archibald [关于缓存的最佳实践][caching-best]

## 将依赖和运行时提取到单独文件

**依赖**

应用的依赖一般不如实际的应用业务代码变化频繁。如果你把它们移动到单独文件，浏览器就可以单独缓存它们，业务代码变化时也无需重新下载它们。

> 🔑 关键术语：在 webpack 术语中，app 代码之外的独立代码文件称为 *chunks*。后面会使用这个名称。

为了把依赖提取到单独的 chunk 中，需要分三个步骤：

1. 将输出文件名称替换为 `[name].[chunkname].js`

```js
/** webpack.config.js */
module.exports = {
  output: {
    filename: '[name].[chunkhash].js'
  }
}
```

当 webpack 编译 app 时，会把 `[name]` 替换为 chunk 的名字。不过不加名称，就需要根据哈希值判断 chunk ，这将十分困难。

2. 将 `entry` 字段转换为对象类型：

```js
/** webpack.config.js */
module.exports = {
  entry: {
    main: './index.js'
  }
}
```

在上面的片段中，`main` 是 chunk 的名字，它会替换步骤 1 中的 `[name]` 部分。现在如果编译 app，这个 chunk 会包含所有的 app 代码，就像以前一样。很快就会变化了。

3. 增加 `CommonsChunkPlugin` 插件：

```js
/** webpack.config.js */
module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      /** 指定 chunk 的名字，它将包含所有依赖，它的名字会替换步骤 1 中的 [name] */
      name: 'vendor',
      /** 用来指定哪些模块可以进入该 chunk 的条件 */
      minChunks: module => module.context && module.context.includes('node_modules')
    })
  ]
}
```

这个插件会把所有路径包含 `node_modules` 的模块移动到一个单独文件，名称为 `vendor.[chunkhash].js`。

经过这些步骤，每次编译会产生两个文件，而不是原来的一个。浏览器可以分别缓存他们，只有在变化后才重新下载。

**webpack 运行时代码**

不幸的是，仅仅提取第三方库是不够的。如果改变 app 的代码：

```js
/** index.js */

// 比如，增加如下代码
console.log('Wat')
```

将会注意到，`vendor` 哈希值也会发生变化。

这是因为当 webpack 打包时，除了模块代码，还有一个[运行时][manifest] - 一小段管理模块执行的代码。当你将代码分离为多个文件时，这段代码开始包括 chunk ID 到对应文件的映射：

```js
/** vendor.e6ea4504.js */
script.src = __webpack_require__.p + chunkId + "." + {
  "0": "2f2269c7f0a55a5c1871"
}[chunkId] + ".js"
```

Webpack 会将运行时打包到最后产生的 chunk 中，在本例中就是 `vendor`。每次任何 chunk 的改变，运行时的这段映射代码都会变化，从而导致整个 `vendor` chunk 变化。

> 在 webpack 3.11.0 没有复现 vendor 哈希值变化，是否已经被修复了？

为了解决它，可以把运行时打包到单独文件中，具体做法是通过 `CommonsChunkPlugin` 创建一个额外空 chunk ：

```js
/** webpack.config.js */
module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: module => module.context &&
        module.context.includes('node_modules'),
    }),

    /** 这个插件必须在 vendor 之后（因为 webpack 会把运行时打包到最后的 chunk） */
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime',
      /** minChunks: Infinity 意味着其中不会包含任何 app 模块 */
      minChunks: Infinity,
    }),
  ],
}
```

之后，每次编译会产生三个文件：

```sh
$ webpack
Hash: ac01483e8fec1fa70676
Version: webpack 3.8.1
Time: 3816ms
                            Asset     Size  Chunks             Chunk Names
   ./main.00bab6fd3100008a42b0.js    82 kB       0  [emitted]  main
 ./vendor.26886caf15818fa82dfa.js    46 kB       1  [emitted]  vendor
./runtime.79f17c27b335abc7aaf4.js  1.45 kB       3  [emitted]  runtime
```

在 `index.html` 中逆向引入这些文件，大功告成：

```html
<!-- index.html -->
<script src="./runtime.79f17c27b335abc7aaf4.js"></script>
<script src="./vendor.26886caf15818fa82dfa.js"></script>
<script src="./main.00bab6fd3100008a42b0.js"></script>
```

延伸阅读

- webpack 指南之[长期缓存][caching]
- webpack 文档之 [webpack 运行时和清单][manifest]
- [物尽其用之 CommonsChunkPlugin][commons-chunk-plugin]

## 内联 webpack 运行时以减少额外 HTTP 请求

为了更好的体验，可以把运行时代码内联到页面，可以帮你减小一个 HTTP 请求。

下面展示具体做法

### 如果使用 HtmlWebpackPlugin 产生页面

如果使用 HtmlWebpackPlugin 插件产生静态页面，[InlineChunkWebpackPlugin][html-webpack-inline-chunk-plugin] 足够了。

```js
/** webpack.config.js */
const HtmlWebpackInlineChunkPlugin = require('html-webpack-inline-chunk-plugin')

module.exports = {
  // ...
  plugins: [
    // ...
    new HtmlWebpackPlugin({
      title: 'Hello Webpack'
    }),
    
    new HtmlWebpackInlineChunkPlugin({
      inlineChunks: ['runtime']
    })
  ]
}
```

### 如果使用自定义后端逻辑产生 HTML

1. 通过设定 `filename` 让运行时的名称固定

```js
/** webpack.config.js */
module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime',
      minChunks: Infinity,
      /** 从此运行时的名称固定为 runtime.js */
      filename: 'runtime.js'
    }),
  ],
};
```

2. 将 `runtime.js` 内容内联到页面中。例如，如果使用 Node.js 和 Express：

```js
/** server.js */
const fs = require('fs')
const runtimeContent = fs.readFileSync('./runtime.js', 'utf-8')

app.get('/', (req, res) => {
  res.send(`
    …
    <script>${runtimeContent}</script>
    …
  `)
})
```

## 懒加载暂时用不到的代码

有时候，有些页面部分优先级较低，但是数量较多：

- 如果加载 YouTube 的视频页面，你关心的是视频而不是评论，此时，视频优先级高于评论。
- 如果打开新闻网站的文章页，你关心的是文章内容，而不是广告。此时，文本比广告重要。

在这些场景，要提升页面首次下载速度，可以首先只下载重要的代码，然后懒加载其他的内容。具体可以使用 [import() 函数][import-func]和[代码分割][code-split]，如下所示：

```js
// videoPlayer.js
export function renderVideoPlayer() { … }

// comments.js
export function renderComments() { … }

// index.js
import { renderVideoPlayer } from './videoPlayer'
renderVideoPlayer()

// …Custom event listener
onShowCommentsClick(() => {
  import('./comments').then((comments) => {
    comments.renderComments()
  })
})
```

`import()` 表明你想动态加载某一特定模块。当 webpack 看到 `import('./module.js')` 时，它会把该模块移动到单独 chunk 中：

```sh
$ webpack
Hash: 39b2a53cb4e73f0dc5b2
Version: webpack 3.8.1
Time: 4273ms
                            Asset     Size  Chunks             Chunk Names
      ./0.8ecaf182f5c85b7a8199.js  22.5 kB       0  [emitted]
   ./main.f7e53d8e13e9a2745d6d.js    60 kB       1  [emitted]  main
 ./vendor.4f14b6326a80f4752a98.js    46 kB       2  [emitted]  vendor
./runtime.79f17c27b335abc7aaf4.js  1.45 kB       3  [emitted]  runtime
```

然后当代码执行到 `import()` 函数时，才下载该 chunk 。

这可以让 `main` bundle 更小，首次下载时间更短。更棒的是，他还能优化缓存 - 如果你只是修改了主 chunk 代码，评论 chunk 不会受到任何影响。

> ✨ 注意：如果你使用 Babel 转译代码，会遇到一个语法报错。因为 Babel 默认不理解 `import()` 的用法。为了避免该错误，可以增加 [`syntax-dynamic-import`][babel-dynamic-import] 插件。

延伸阅读

- webpack 文档之 [`import()` 函数][import-func]
- JavaScript 提案之[实现 `import()` 语法][proposal-dynamic-import]

## 将代码拆分为路由和页面

如果你的页面有多个路由或页面，但是只有一个单一 JS 文件（一个 `main` chunk），很可能每次请求时都会带上多余的字节。比如，当用户访问你的首页时，他们其实不需要其他页面的代码，但是也不得不下载。如果用户经常只访问首页，而且你修改了文章页的代码，webpack 就会让整个 bundle 失效，用户就不得不下载整个 app 代码。

如果我们将代码拆分为多个页面（如果是单页，就拆分为多个路由），用户仅需要下载相关的代码。另外，浏览器会更好的缓存 app 代码：如果你修改了首页代码，webpack 仅会让相应的 chunk 失效。

### 对于单页应用

若要将单页 app 按路由拆分，可以使用 `import()`（参见上一小节）。如果你使用了框架，框架很可能已经自带解决方案：

- `react-router` 文档之[“代码分割”][react-router]（适用于 React）
- `vue-router` 文档之[“懒加载路由”][vue-router]（适用于 Vue.js）

### 对于传统的多页应用

为了将传统应用按页拆分，可以使用 webpack 的 [`entry points`][entry-points]。如果你的 app 包含三种类型页面：首页、文章页和用户账号页，它的入口可以这么设置：

```js
/** webpack.config.js */
module.exports = {
  entry: {
    home: './src/Home/index.js',
    article: './src/Article/index.js',
    profile: './src/Profile/index.js'
  },
}
```

对每个入口文件，webpack 都会构建一个单独的依赖树，并产生一个响应的 bundle，其中只包含该入口文件引用的模块：

```
$ webpack
Hash: 318d7b8490a7382bf23b
Version: webpack 3.8.1
Time: 4273ms
                            Asset     Size  Chunks             Chunk Names
      ./0.8ecaf182f5c85b7a8199.js  22.5 kB       0  [emitted]
   ./home.91b9ed27366fe7e33d6a.js    18 kB       1  [emitted]  home
./article.87a128755b16ac3294fd.js    32 kB       2  [emitted]  article
./profile.de945dc02685f6166781.js    24 kB       3  [emitted]  profile
 ./vendor.4f14b6326a80f4752a98.js    46 kB       4  [emitted]  vendor
./runtime.318d7b8490a7382bf23b.js  1.45 kB       5  [emitted]  runtime
```

因此，如果只有文章页使用 Lodash，`home` 和 `profile` bundle 就不会包含它，用户也无需在访问首页时下载整个库。

但是多依赖树也有缺点。如果两个入口都用到了 Lodash，而且没有尚未将依赖移动到 `vendor` bundle，两个入口就会同时包含一份 Lodash 副本。为了解决这个问题，可以使用 `CommonsChunkPlugin` - 它会将公共的依赖移动到独立的文件中：

```js
/** webpack.config.js */
module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      /** 包含公共依赖的 chunk 名字 */
      name: 'common',

      /** 
      * 该插件会把引用数大于 2 的模块移动到公共文件 
      * （注意插件会分析所有 chunk，而不是仅仅入口模块）
      */
      minChunks: 2,    // 2 is the default value
    }),
  ],
}
```

需要多次尝试设置 `minChunks` 的值，才能找到最佳数值。通常，希望其数值越小越好，但会随着 chunk 数量增多而变大。比如，chunk 为 3 时，`minChunks` 可以为 2，但是对于 30 个 chunk，数值可能会变为 8 - 因为如果你将其控制在 2，会有太多的模块进入 common 文件，导致其体积急剧膨胀。

延伸阅读

- webpack 文档之 [entry points 概念][entry-points]
- webpack 文档之 [CommonsChunkPlugin][plugins-commons]
- [物尽其用之 CommonsChunkPlugin][commons-chunk-plugin]

## 使模块 id 更稳定

当构建代码时，webpack 会给每个模块分配一个 ID。这些 ID 会用在后来的 `require()` 语句中。通常可以在构建输出中看到模块 ID，就在模块路径之前。

默认情况下，ID 通过一个计数器计算（比如，第一个模块是 0，第二个模块是 1，等等）。这种处理方式的问题在于如果你增加了新的模块，它可能出现在模块列表中间，使后面的模块 ID 全部变化。

ID 的变化会让所有依赖它的 chunk 失效 - 即使它们的实际代码没有变化。为了解决这个问题，可以使用 [`HashedModuleIdsPlugin`][hashed-module-ids-plugin] 计算模块 ID。它会用模块路径的哈希值代替计数器算出的 ID。

通过这种方式，只有重命名或者移动模块路径时，才会改变模块的 ID。新的模块加入不会影响其他模块的 ID。

为了开启该模块，可以将其加入到 `plugins` 选项：

```js
/** webpack.config.js */
module.exports = {
  plugins: [
    new webpack.HashedModuleIdsPlugin(),
  ],
}
```

延伸阅读

- webpack 文档之 [`HashedModuleIdsPlugin`][hashed-module-ids-plugin]

## 总结

- 缓存 bundle，通过改变名称更新缓存
- 将 bundle 拆分为 app 代码，厂商代码和运行时代码
- 内联运行时代码，节省 HTTP 请求
- 使用 `import()` 懒加载不重要的代码
- 将代码按路由或页面拆分，防止加载非必须的代码

## REF

- [Make use of long-term caching][google], by Ivan Akulov, Google Developers
- [Caching best practices & max-age gotchas][caching-best], by Jake Archibald, 2016/04/27

[google]: https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching
[step1]: ./decrease-frontend-size.md
[caching-best]: https://jakearchibald.com/2016/caching-best-practices/
[hash-bug]: https://github.com/webpack/webpack/issues/1479
[html-webpack-plugin]: https://github.com/jantimon/html-webpack-plugin
[webpack-manifest-plugin]: https://github.com/danethurber/webpack-manifest-plugin
[manifest]: https://webpack.js.org/concepts/manifest/
[caching]: https://webpack.js.org/guides/caching/
[commons-chunk-plugin]: https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318
[html-webpack-inline-chunk-plugin]: https://github.com/rohitlodha/html-webpack-inline-chunk-plugin
[import-func]: https://webpack.js.org/api/module-methods/#import-
[code-split]: https://webpack.js.org/guides/code-splitting/
[babel-dynamic-import]: https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import
[proposal-dynamic-import]: https://github.com/tc39/proposal-dynamic-import
[react-router]: https://reacttraining.com/react-router/web/guides/code-splitting
[vue-router]: https://router.vuejs.org/en/advanced/lazy-loading.html
[entry-points]: https://webpack.js.org/concepts/entry-points/
[plugins-commons]: https://webpack.js.org/plugins/commons-chunk-plugin/
[hashed-module-ids-plugin]: https://webpack.js.org/plugins/hashed-module-ids-plugin/