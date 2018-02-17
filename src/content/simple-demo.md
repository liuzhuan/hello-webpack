# 最简样例

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