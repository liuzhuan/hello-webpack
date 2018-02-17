# Manifest

webpack 构建的应用或网站，主要有以下三种代码：

1. 你或你的团队编写的代码
2. 你代码依赖的第三方类库
3. webpack 运行时和指导所有模块如何交互的 *manifest* 清单

本章节主要关注 webpack 运行时和 manifest 清单。

## 运行时 Runtime

当你的应用在浏览器中运行时，各模块会通过 webpack 运行时和 manifest 连接。它包括模块的加载和解析逻辑，包括已下载模块的互动和未加载模块的懒加载。

## Manifest

一旦你的应用在浏览器中运行，它就会变为 `index.html`，一些 bundles，一些其他类型的素材。你曾精心设计的 `src/` 目录不见了，那么 webpack 是如何管理所有模块之间的交互的？这里用到了 manifest 

当编译器进入，解析，映射你的应用时，它会详细记录所有模块的信息。这些数据集合称为“manifest”，会被运行时用来解析和加载模块。不管你使用哪种模块语法，这些 `import` 或 `require` 语句会变为 `__webpack_require__` 方法，指向模块的标识符。借助 manifest 数据，运行时会知道何处获取特定标识符指向的模块。

## 问题

大部分情况下，在你不知道的情况下，运行时会把所有这些工作搞定。但是一旦你决定使用浏览器缓存提高应用的效率，理解这些过程会必不可少。

文件名的内容哈希可以通知浏览器什么时候文件缓存失效。但是你开始使用这个特性后，很快会发现一些有趣的特性。即使一些文件内容没有变化，哈希值还是会变化。这些变化是注入的运行时和 manifest 造成的，他们每次编译都会变化。

## 参考资料

- [The Manifest](https://webpack.js.org/concepts/manifest/) - webpack.js.org
- [Guides - Output Management](https://webpack.js.org/guides/output-management/) - webpack.js.org
- [Separating a Manifest | survivejs](https://survivejs.com/webpack/optimizing/separating-manifest/)
- [Guides - Caching](https://webpack.js.org/guides/caching/) - webpack.js.org
- [Predictable Long Term Caching with Webpack | medium](https://medium.com/webpack/predictable-long-term-caching-with-webpack-d3eee1d3fa31)