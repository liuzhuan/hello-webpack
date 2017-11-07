## 素材管理 Asset Management

webpack 最酷的特性是可以包含任意类型的文件。

### 加载 CSS

为了在 JavaScript 模块中引入 CSS 文件，你需要安装 `style-loader` 和 `css-loader`，并在 `module` 配置项中设置：

```
npm install --save-dev style-loader css-loader
```

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ]
  }
}
```

这样就可以在 js 文件中引入 CSS 文件：

**src/index.js**

```js
import './style.css'
```

### REF

- https://webpack.js.org/guides/asset-management/