---
title: 优化webpack打包时间
date: 2020-12-07 00:38:45
tags:
  - webpack

categories:
  - webpack
---

在开发一个项目的时候，一开始webpack进行编译的速度都是很快的，因为我们引入的依赖很少。但是，当你的项目越来越臃肿的时候，打包的时间将大幅度的上升，有时候达到几分钟这种完全不能接受的地步。这时候，我们就应该开始着手对于webpack的打包流程进行优化了。

1. 分析依赖

想要知道什么东西影响了打包的时间，那么我们当然需要了解我们打包出来的文件的构成。是哪些依赖导致的我们打包速度变慢。这时候，我们就要用到 webpack-bundle-analyzer 这个插件了。在使用了这个插件后，会自动打开一个网页，里面的图片就是我们打包结果的构成。如下图所示。

![](https://floatdai-blog.oss-cn-beijing.aliyuncs.com/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202020-12-14%20002309%20%281%29.png?Expires=1607880388&OSSAccessKeyId=TMP.3Kk17ATZT8SrodLCzaxq2K4KsDXf7CogQeahPaTN9Nne1XCEwLVvNd8QbT9dWiytBkexfRVF7MptsxWDHDpnjhitAMQp9i&Signature=S7Lek%2FUnHyxEQuRLRrtgsw1UZhE%3D&versionId=CAEQDRiBgICSitnzshciIDc0Y2YxNTg2NTg3ZDRlZTNiMjhmN2U1ZDQzYWViMjMy&response-content-type=application%2Foctet-stream)

显而易见的，我们可以发现，element 和 echarts 这两个库占用了大量的体积，我们可以对此进行优化。

上面的插件是用来分析每个依赖的体积的。但是，我们要关心的不止是体积，我们关注点的根源应该是时间。所以，我们需要一个东西来分析所有的依赖打包所花的时间。这时候，我们就要用到 speed-measure-webpack-plugin 这个东西了。这个东西的引入也很简单。

```
// webpack file
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin")
const smp = new SpeedMeasurePlugin()

module.exports = smp.wrap(webpackConfig)
```

使用如上所示的代码把 webpack 配置包裹在这个函数中即可。

最后，我们运行命令进行打包可以看到如下图所示的结果。他们分别列出了每一个loader运行所花的时间。我们可以针对这些数据进行各自优化。

![](https://floatdai-blog.oss-cn-beijing.aliyuncs.com/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202020-12-14%20004832%20%281%29.png?Expires=1607881788&OSSAccessKeyId=TMP.3Kk17ATZT8SrodLCzaxq2K4KsDXf7CogQeahPaTN9Nne1XCEwLVvNd8QbT9dWiytBkexfRVF7MptsxWDHDpnjhitAMQp9i&Signature=K0HZByEYACZvDny67JF1A5PQ%2BBA%3D&versionId=CAEQDRiBgMDU9YL0shciIDBhMTgzYjA3YzA4YjQ2MzNiMzNhNDQ5MThkODA0YWM1&response-content-type=application%2Foctet-stream)

2. cache-loader

说到优化打包时间，那么我们第一时间想到的就是利用缓存，利用空间换时间了。第一次打包时在磁盘中存储一些打包后的结果，之后每次打包都可以利用这个结果那是极好的了。

cache-loader 就是用来做这件事情的东西。不过,第一次打包的时候可能要划更多的时间，因为需要打包一些需要缓存的内容。第二次之后就可以使用这些缓存的内容加快打包速度了。

配置他很简单，把他放在其他loader之前即可。

```
{
  test: /\.vue$/,
  use: [
    "cache-loader",
    "vue-loader",
  ],
}
```

我们仍然使用之前的项目进行测试。我们对于其中标红的两个包，babel-loaer 和 vue-loader 分别都配置了 cache-loader。第一次打包花了78s，第二次打包花了60s。

而当我们不使用 cache-loader 的时候则花了77s。

可以看到，使用了 cache-loader 后，第二次开始的打包时间有了明显的提升。

3. HardSourceWebpackPlugin

HardSourceWebpackPlugin 同样是一个使用缓存来进行加速打包的一个插件。他的缓存默认被存在 /.cache/hard-source 文件夹中。第一次打包他将花费更多的时间以用于进行缓存。第二次打包则有显著的速度提升。

我们通用使用之前的项目进行测试。第一次打包花了149秒，第二次打包花了45秒。相比于cache-loader的时间有了不小的提升。

我们再尝试一下同时使用 cache-loader 和 HardSourceWebpackPlugin 进行打包。第一次打包花了156秒，第二次打包花了44秒。和单独使用 HardSourceWebpackPlugin 打包的速度并没有什么区别。

当然，在配置这个插件的时候还是有很多坑的。这些在这个插件的 README 中的 [troubleshooting](https://www.npmjs.com/package/hard-source-webpack-plugin#troubleshooting)都有提到。

4. external

external 是 webpack 中的一个配置项。这个配置项的功能是在打包的时候不对某些包进行打包。而是通过 CDN 的方式来引入这些包。这就是通过减少打包的内容来提高打包的时间的一种配置。

示例配置如下。

```
// in webpack config
external: {
  jquery: "jQuery",
}

// in code
import $ from "jquery"
```

这时候，这个 $ 就相当于引入 jQuery 了。

5. webpack dll

dll 是 Windows 中的一个概念。这里用维基百科上话来解释一波。

> 所谓动态链接，就是把一些经常会共享的代码（静态链接的OBJ程序库）制作成DLL档，当可执行文件调用到DLL档内的函数时，Windows操作系统才会把DLL档加载存储器内，DLL档本身的结构就是可执行档，当程序有需求时函数才进行链接。通过动态链接方式，存储器浪费的情形将可大幅降低。静态链接库则是直接链接到可执行文件。
> DLL的文件格式与视窗EXE文件一样——也就是说，等同于32位视窗的可移植执行文件（PE）和16位视窗的New Executable（NE）。作为EXE格式，DLL可以包括源代码、数据和资源的多种组合。
> 在更广泛的意义上说，任何同样文件格式的电脑文件都可以称作资源DLL。这样的DLL的例子有扩展名为ICL的图标库、扩展名为FON和FOT的字体文件。

所以，其实 dll 其实也是一种缓存的形式。在 webpack 中，他的使用方式是事先把一些第三方的库打包成一个个dll文件，在打包的时候就可以跳过这些文件进行打包。直接使用这些文件代码即可运行。

让我们来看看怎么使用这个东西。

首先要有一个生成 dll 文件的配置文件。

```
const vendors = [
  "vue",
  "vue-router",
  "vuex",
]

module.exports = {
  entry: {
    vendor: vendors,
  },
  output: {
    path: path.resolve(__dirname, "../dll"),
    filename: "[name]_[fullhash].js",
    library: "[name]_[fullhash]",
  },
  plugins: [
    new webpack.DllPlugin({
      context: path.resolve(__dirname, "../"),
      name: "[name]_[fullhash]",
      path: path.join(__dirname, "manifest.json"),
    })
  ],
}
```

使用 webpack 运行上面的配置文件即可生成 dll 文件。生成的 dll 文件全都生成在了 dll 文件夹中。输出的配置文件则是 manifest.json

```
new webpack.DllReferencePlugin({
  context: path.resolve(__dirname, "../"),
  manifest: path.resolve(__dirname, "manifest.json"),
})
```

想要使用的这些 dll 文件只要在 webpack 配置中的 plugin 加上一个新的 plugin 即可。在里面配置一下之前生成的 dll 输出的配置文件 manifest.json 即可。

6. 开启多进程

- terser-webpack-plugin 可以开启多进程选项。这是一个压缩JS代码的差距。

- thread-loader: 对于那些比较耗时的loader可以配置一下 thread-loader 开启多进程。配置也非常简单，只要把他放在想要配置的loader之前即可。但是进程的启动是需要时间的，所以最好只在耗时的loader上配置 thread-loader。否则可能会话更多的时间。

7. webpack 5

每次webpack 的升级都带来了打包性能不小的提升，所以升级你的webpack，一定可以得到不小的惊喜。
