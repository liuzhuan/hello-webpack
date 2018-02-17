# Hello Webpack

![Webpack Logo](./assets/logo-on-white-bg.png)

webpack 是德国程序员 *Tobias Koppers* 开发的一个前端模块打包器。

Tobias 开发 Java 期间，使用过谷歌的 GWT (Google Web Toolkit)。GWT 代码分割特性让 Tobias 念念不忘。后来 Tobias 转行开发 JavaScript，发现当时的 JavaScript 打包工具缺少代码分割功能。于是他向 webmake（当时的 JS 打包工具）作者提交了[一个建议](https://github.com/medikoo/modules-webmake/issues/7)，结果没通过。后来他决定自己开发一个工具，webpack 就产生了。

2012年5月10日，Tobias 在 webpack 库中[第一次提交 2e1460](https://github.com/webpack/webpack/commit/2e1460036c5349951da86c582006c7787c56c543)。

## 最简样例

首先编写入口文件 `app.js` 和模块文件 `bar.js`:

```js
// app.js
import bar from './bar'
bar()
```

```js
// bar.js
export default function bar() {}
```

接着创建配置文件 `webpack.config.js`，内容如下：

```js
module.exports = {
  entry: './app.js',
  output: {
    filename: 'bundle.js'
  }
}
```

最后，在命令行运行 `webpack` 即可，随后在当前目录生成 `bundle.js` 。