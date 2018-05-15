# `output.publicPath`

这个命令用来设定外部资源（图片、文件等）的基础路径，若设置错误，将会遭遇 404 错误。

这个选项的值会位于每个 URL 前，因此，大部分情况下会以 `/` 结尾。

默认值是 `""`。

## 使用场景

在真实应用中有一些使用场景，这个特性十分有效。实际上，`output.path` 目录下的每个文件都会被 `output.publicPath` 引用。包括子块（通过 code splitting 创建）和其他的资源。

TODO

## REF

- [Public Path - guides][guides]
- [output.publicPath][config]

[guides]: https://webpack.js.org/guides/public-path/
[config]: https://webpack.js.org/configuration/output/#output-publicpath