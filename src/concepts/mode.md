# 模式 Mode

[online reading](https://webpack.js.org/concepts/mode/)

1. 可将 `mode` 参数设为 `development`, `production`, `none` 三者之一，方便 webpack 在不同场景下进行合适的优化策略。
2. 默认值是 `production`。
3. 也可以通过命令行参数设定模式。`webpack --mode=production`
4. 当 `mode = development` 时，会把 `DefinePlugin` 中的 `process.env.NODE_ENV` 设为 `development`，同时启用 `NamedChunksPlugin` 和 `NamedModulesPlugin` 插件。
5. 当 `mode = production` 时，会把 `DefinePlugin` 中的 `process.env.NODE_ENV` 设为 `production`，同时启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin`, `TerserPlugin` 插件。

## 相关代码

基本用法

```js
module.exports = {
    mode: 'production'
};
```

如果需要根据模式不同改变 webpack 配置的行为，可以导出函数：

```js
var config = {
    entry: './app.js',
    // ...
}

module.exports = (env, argv) => {
    if (argv.mode === 'development') {
        config.devtool = 'source-map';
    }

    if (argv.mode === 'production') {
        // ...
    }

    return config;
};
```