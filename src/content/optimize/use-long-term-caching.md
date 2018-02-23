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