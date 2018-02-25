# 模块热更新的概念

热更新可以在应用运行时替换、增加或删除模块，无须重载页面。这样可以极大提高开发速度，具体表现在以下几点：

- 重载页面会导致应用状态丢失
- 只更新改变的部分可以提升宝贵的开发时间
- 更快的样式调整速度，几乎可以与浏览器的 debugger 相媲美

## 工作原理

我们从几个不同角度来理解 HMR 是如何工作的...

### 从应用的角度看

以下步骤允许模块子在应用中换入换出：

1. 应用向 HMR 运行时请求更新内容
2. 运行时异步下载更新后，通知应用
3. 应用请求运行时安装更新
4. 运行时同步安装更新

你可以设置 HMR 让这个过程自动化，也可以在更新发生前加入用户交互。

### 从编译器的角度看

除了普通素材外，编译器需要抛出 `update` 事件，将旧版本更新为新版本。`update` 包含两个部分：

1. 更新的 [manifest][manifest] (JSON)
2. 一个或多个更新的 chunks（JavaScript）

manifest 包含新的编译哈希值和所有已更新 chunks 列表。每个 chunk 包含所有已更新模块的新代码（或者一个表示模块已被删除的标示位）。

编译器确保模块 ID 和 chunk ID 在编译期间是一致的。它通常将这些 ID 放在内存中（比如，利用 [webpck-dev-server][dev-server]），但是也可能将其放到 JSON 文件中。

### 以模块的角度看

HMR 是一个可选特性，它只会影响包含 HMR 代码的模块。以 `style-loader` 增加样式补丁为例。为了让更新的样式生效，`style-loader` 需要实现 HMR 接口；当它通过 HMR 得到更新，就会用新代码替换旧代码。

同样的，当在模块实现 HMR 接口时，你可以描述更新模块时做什么操作。然而，大多数场景下，无需在每个模块编写 HMR 代码。如果模块没有 HMR 处理函数，更新就会冒泡上浮。这意味着，一个处理函数就可以更新一个完整的模块树。如果树中的一个模块更新，整个依赖就会被重新加载。

See the HMR API page for details on the module.hot interface.

参见 [HMR API 页][api]来获取 `module.hot` 接口的详情。

### 以运行时的角度看

这里的描述偏向技术。如果你对内部细节不感兴趣，可以翻阅 [HMR API 页][api]或 [HMR 指南][guides]。

对于模块系统运行时，会增加额外代码追踪代码的 `parents` 和 `children`。在管理方面，运行时支持两个方法：`check` 和 `apply`。

`check` 向更新清单（update manifest）发出一个 HTTP 请求。如果请求失败，不会有任何更新。若成功，已更新 chunk 列表会与当前已加载 chunk 列表比对。对于每个已加载 chunk，会下载响应的更新 chunk 。所有的模块更新储存在运行时中。当所有的更新模块加载完毕，准备好应用时，运行时切换到 `ready` 状态。

`apply` 方法将所有已更新模块标记为失效。对每个失效模块，需要在模块或其父级模块有一个更新回调函数。否则，失效标记就会冒泡上浮，将所有父级也失效。冒泡一直上浮，直到抵达 app 入口模块或遇到一个更新回调函数。如果冒泡来自入口模块，这个过程就会失败。

最后，所有的失效模块都会被清除（通过清除回调函数）卸载。接着更新当前哈希值，调用所有的 `accept` 回调函数。运行时切换回 `idle` 状态，一切恢复平静。

## 开始

HMR 可以在开发过程中代替 LiveReload 。[webpack-dev-server][dev-server] 支持 `hot` 模式，其中它会尝试使用 HMR 更新，然后才会尝试更新整个页面。可以参阅[模块热更新][guides]了解详情。

## REF

- [Hot Module Replacement][hmr] - webpack.js.org

[hmr]: https://webpack.js.org/concepts/hot-module-replacement/
[manifest]: https://webpack.js.org/concepts/manifest/
[dev-server]: https://webpack.js.org/configuration/dev-server/
[api]: https://webpack.js.org/api/hot-module-replacement/
[guides]: https://webpack.js.org/guides/hot-module-replacement