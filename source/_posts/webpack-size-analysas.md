---
title: 基于 Webpack 的应用包体尺寸优化
category: webpack
tags: webpack
---

最近我在构建一个基于 React 的单页应用，当我用Google TestMySite来检测自己的站点时，它的反馈是加载时间过长，因此我开始考虑如何优化初次下载的包体大小。优化应用包体的第一步就是检视当前的包体组成，判断其中哪些依赖时必须的，我们在 Webpack 的回显中可以看到当前的包体大小：
<!--more-->
```bash
$ webpack -p --progress
Hash: dbce3735c9520e2dc682
Version: webpack 1.14.0
Time: 54264ms
            Asset     Size  Chunks             Chunk Names
    dist/index.js  3.29 MB       0  [emitted]  main
dist/index.js.map  13.7 MB       0  [emitted]  main
   [0] multi main 40 bytes {0} [built]
    + 1374 hidden modules
```
# 分析包体依赖

这里我们使用webpack-bundle-analyzer来分析 Webpack 生成的包体组成并且以可视化的方式反馈给开发者。我们可以使用npm来安装该插件：
```js
$ npm install --save-dev webpack-bundle-analyzer
```
然后我们需要修改webpack.config.js来引入该插件：
```js
var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
// ...
plugins: [new BundleAnalyzerPlugin()]
// ...
```阿达撒大声地
然后我们照常使用 Webpack 编译之后，可视化的结果会被展示在http://localhost:8888/，你大概可以看到如下的交互界面：

这个交互插件应该能帮你分析哪些依赖占据了主要的包体大小，这也提醒我们在引入某个功能的时候，应该只引入需要的模块，以 Lodash 为例：
```js
//1. Import the entire lodash library and add it to the bundle
import lodash from 'lodash'
lodash.groupBy(rows, 'id')

//2. Import only the required function from lodash
import groupBy from 'lodash/groupBy'
groupBy(rows, 'id')
```
# 设置合适的 Node 环境变量

设置合适的环境变量能够帮助 Webpack 更好地去压缩处理依赖中的代码，我们需要在生产环境中将NODE_ENV设置为production:
```js
plugins: [...
    new webpack.DefinePlugin({
             'process.env.NODE_ENV': '"production"'
    }),
..]
```
# 使用最小化的 SourceMap

当我们在生产环境下组合压缩 JavaScript 文件时，Webpack 会为我们生成某个 SourceMap 文件来映射源文件的内容。在开发环境中我们经常会将devtool设置为eval，这样会将大量的代码信息打包到输出包体中从而提升编译速度。而开发环境中我们可以将该项设置为eval-source-map或者cheap-module-source-map，详细介绍参考这里，我们比较切换前后的包体大小可以发现缩小了将近 1MB 的内容：
```bash
$ webpack -p --progress
Hash: 68a52fddbcc2898a5899
Version: webpack 1.14.0
Time: 29757ms
            Asset     Size  Chunks             Chunk Names
    dist/index.js  1.71 MB       0  [emitted]  main
dist/index.js.map  464 bytes       0  [emitted]  main
   [0] multi main 40 bytes {0} [built]
    + 1365 hidden modules
```
# 其他常用插件

这里我列举几个常用的能够用于减少包体大小的插件，我们可以根据项目需求选择性的使用：

compression-webpack-plugin:该插件能够将资源文件压缩为.gz文件，并且根据客户端的需求按需加载。

dedupeplugin:抽取出输出包体中的相同或者近似的文件或者代码，可能对于 Entry Chunk 有所负担，不过能有效地减少包体大小。

uglifyjsplugin:压缩输出块的大小，可以参考官方文档。

ignoreplugin:用于忽略引入模块中并不需要的内容，譬如当我们引入moment.js时，我们并不需要引入该库中所有的区域设置，因此可以利用该插件忽略不必要的代码。
```js
...
var CompressionPlugin = require("compression-webpack-plugin");
...
let config = {
  entry: path.join(__dirname, '../app/index'),
  cache: false,
  devtool: 'cheap-module-source-map',
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"production"'
    }),
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.UglifyJsPlugin({
      mangle: true,
      compress: {
        warnings: false, // Suppress uglification warnings
        pure_getters: true,
        unsafe: true,
        unsafe_comps: true,
        screw_ie8: true
      },
      output: {
        comments: false,
      },
      exclude: [/\.min\.js$/gi] // skip pre-minified libs
    }),
    new webpack.IgnorePlugin(/^\.\/locale$/, [/moment$/]),
    new webpack.NoErrorsPlugin(),
    new CompressionPlugin({
      asset: "[path].gz[query]",
      algorithm: "gzip",
      test: /\.js$|\.css$|\.html$/,
      threshold: 10240,
      minRatio: 0
    })
    ...
  ],
```
引入该插件后包体的体积会受到进一步的压缩：

```bash
$ webpack -p --progress
Hash: 68a52fddbcc2898a5899
Version: webpack 1.14.0
Time: 29757ms
            Asset     Size  Chunks             Chunk Names
    dist/index.js  1.54 MB       0  [emitted]  main
dist/index.js.gz   390 KB        0  [emitted]  main
dist/index.js.map  464 bytes     0  [emitted]  main
   [0] multi main 40 bytes {0} [built]
    + 1365 hidden modules
```