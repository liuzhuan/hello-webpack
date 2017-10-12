# 版本迁移

webpack 1 和 webpack 2 有较大的差别，2 到 3 的差别要小的多，因此以下内容主要针对 1 到 2 的迁移。

## `resolve.root`, `resolve.fallback`, `resolve.moduleDirecotories`

这个选项统一被 `resolve.modules` 代替：

```diff
resolve {
-  root: path.join(__dirname, 'src')
+  modules: [
+    path.join(__dirname, 'src'),
+    'node_modules'
+  ]
}
```

## 参考文献

- [Migrating Versions](https://webpack.js.org/guides/migrating/) - webpack.js.org