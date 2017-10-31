## 级联 loaders

同 webpack 1 一样，loaders 可以串联使用。通过 `rule.use` 配置项，`use` 可以设定为一个 loaders 数组。在 webpack 1 中，loaders 通常使用 `!` 级联。这种风格仅仅在遗留选项 `module.loaders` 可以使用：

```diff
module: {
-  loaders: [{
+  rules: [{
    test: /\.less$/,
-    loader: "style-loader!css-loader!less-loader"
+    use: [
+      "style-loader",
+      "css-loader",
+      "less-loader"
+    ]
  }]
}
```

### REF
- https://webpack.js.org/guides/migrating/#chaining-loaders