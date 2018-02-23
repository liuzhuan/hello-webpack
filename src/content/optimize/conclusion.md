# 总结

- 删减无效字节。压缩所有资源，丢弃多余代码，添加依赖时要谨慎。
- 根据路由拆分代码。只加载目前最重要的代码，懒加载其余资源。
- 缓存代码。一些代码无需频繁更新，将其放入独立文件，让其仅在必须时才重新下载。
- 追踪代码体积。使用 `webpack-dashboard` 和 `webpack-bundle-analyzer` 等工具，实时监控 app 体积。每隔一段时间，就监测 app 的性能。

除了 webpack 还有其他工具可以让你的代码运行更快。考虑让你的应用变为 [PWA][pwa]，获得更佳用户体验，或者使用 [Lighthouse][lighthouse] 等自动化工具，来获得优化建议。

别忘了 [webpack 官方文档][guides]，那里有很多宝贵资源。

## REF

- [Conclusion][google]

[google]: https://developers.google.com/web/fundamentals/performance/webpack/conclusion
[pwa]: https://developers.google.com/web/progressive-web-apps/
[lighthouse]: https://developers.google.com/web/tools/lighthouse/
[guides]: https://webpack.js.org/guides/