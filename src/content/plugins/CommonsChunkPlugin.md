## CommonsChunkPlugin

`CommonsChunkPlugin` 可以把多个入口文件公用的模块提取出来，放置到一个单独的文件（也称之为 chunk）。这样只需下载一次公用模块，就可以保留在浏览器缓存中，提升页面加载速度。

```js
new webpack.optimize.CommonsChunkPlugin(options)
```

### 选项

```js
{
  // 公用 chunk 的 chunk 名。
  name: string,
  names: string[], 
  // 公用 chunk 的文件名模板。可以包含与 `output.filename` 类似的占位符。
  filename: string, 
  // 只有多于这个数字的 chunks 包含一个 module，才会将这个 module 移入公用 chunk。
  minChunks: number|Infinity|function(module, count) -> boolean,
  // 通过代码块名称选择源代码块。
  chunks: string[],
  // 如果是 `true`，则选择所有的公用代码块子元素
  children: boolean, 
  // 如果是 `true`，则选择所有功用代码块的后台元素
  deepChildren: boolean,
  // 如果是 `true`，则一个新的异步模块被创建，作为 `options.name` 的子元素和 `options.chunks` 的兄弟元素
  async: boolean|string,
  // 所有公用代码块最小体积
  minSize: number,
}
```

### 样例

创建入口的公用 chunk 

```js
new webpack.optimize.CommonsChunkPlugin({
  name: 'commons',
  filename: 'commons.js',
})
```

必须在入口文件之前加载产生的 chunk：

```html
<script src="commons.js" charset="utf-8"></script>
<script src="entry.bundle.js" charset="utf-8"></script>
```

明确的第三方 chunk

```js
entry {
  vendor: ['jquery', 'other-lib'],
  app: './entry'
},
plugins: [
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks: Infinity,
  })
]
```

### 参考资料

- https://webpack.js.org/plugins/commons-chunk-plugin/
- https://github.com/webpack/webpack/blob/master/lib/optimize/CommonsChunkPlugin.js