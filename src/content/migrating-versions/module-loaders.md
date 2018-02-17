## `module.loaders` 变为 `module.rules`

旧版 loader 配置替换为一个更强大的 rules 系统，它允许配置 loaders。为了兼容性，旧版 `module.loaders` 语法依然有效。新版命名规则理解更简单，建议升级到 `module.rules` 选项。

```diff
module: {
-  loaders: [
+  rules: [
    {
      test: /\.css$/,
-      loaders: [
-        "style-loader",
-        "css-loader?modules=true"
+      use: [
+        {
+          loader: "style-loader"
+        },
+        {
+          loader: "css-loader",
+          options: {
+            modules: true
+          }
+        }
      ]
    },
    {
      test: /\.jsx$/,
      loader: "babel-loader", // Do not use "use" here
      options: {
        // ...
      }
    }  
  ]
}
```

### REF
- https://webpack.js.org/guides/migrating/#module-loaders-is-now-module-rules