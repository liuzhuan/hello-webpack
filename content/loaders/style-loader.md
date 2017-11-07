## style-loader

### 安装

```
npm install style-loader --save-dev
```

### 用法 Usage

建议结合使用 `style-loader` 和 `css-loader`

**component.js**

```js
import style from './file.css'
```

**webpack.config.js**

```js
{
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          { loader: 'css-loader' },
        ]
      }
    ]
  }
}
```

当使用本地作用域的 CSS 时，模块会导出产生的标识符：

**component.js**

```js
import style from './file.css'
style.className === 'z849f98xxx'
```

### REF

- https://webpack.js.org/loaders/style-loader/