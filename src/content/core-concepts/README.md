# 核心概念

身为打包器，webpack 处理应用程序时，会递归地生成一个依赖图（`dependency graph`），其中涵盖了所有模块，接着会把模块打包为一定数量的包裹（`bundles`）。

它的配置项很多，但核心概念只有四个：`entry`, `output`, `loaders` 和 `plugins`。

下面对它们做简要介绍。