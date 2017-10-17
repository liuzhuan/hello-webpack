## clean-webpack-plugin

构建之前清除构建目录的 webpack 插件

### 安装

```
npm i clean-webpack-plugin --save-dev
```

### 用法

```js
const CleanWebpackPlugin = require('clean-webpack-plugin')

// webpack config
{
  plugins: [
    new CleanWebpackPlugin(paths[, {options}])
  ]
}
```

使用样例如下：

```js
const CleanWebpackPlugin = require('clean-webpack-plugin')

let pathsToClean = [
  'dist',
  'build'
]

let cleanOptions = {
  root: '/full/webpack/root/path',
  exclude: ['shared.js'],
  verbose: true,
  dry: false
}

module.exports = {
  plugins: [
    new CleanWebpackPlugin(pathsToClean, cleanOptions)
  ]
}
```

### 源码解析

依赖 `rimraf` 库进行底层操作。

### 参考资料

- [clean-webpack-plugin](https://github.com/johnagan/clean-webpack-plugin)