---
title: Hugo with Vue
date: 2020-09-18 14:31:42
tags:
  - Hugo
  - Vue

categories:
  - HTML
---

本文主要讲述如何在 Hugo 中使用 Vue 进行开发一些复杂交互的网站。

## Hugo

Hugo 是一个 static site generator(静态网站生成)的框架。静态网站生成指的是利用一些模板文件，预先将所有的HTML文件生成好，挂载在服务端的一种技术。利用 Hugo 你就可以生成一系列网页。但是由于这种框架都是只基于简单的 HTML，JS，CSS，而不涉及任何其他的框架，所以如果要开发一些比较复杂的网站的时候就会变得比较困难。这时候，利用一些现代的框架（如Vue，React，Angular等等）来简化我们的开发。这里我们就以 Vue 为例来看看怎么使用 Vue 来配合 Hugo 进行开发。

## 前提知识

本文将假设读者已经了解以下知识：

- Hugo 的基本知识（文件夹结构，编译流程等等）
- Vue 的基本知识
- webpack 配置基础

## 基本思路

想要使用Vue进行开发，就先思考一下，我们平时使用的Vue编译后的结果是什么样子的。只要我们模仿一下这个编译后的结果，就可以使用Vue进行开发了。

首先，我们肯定是要分两种环境进行考虑的，一种是开发环境，一种是生产环境。开发环境使用热加载开发简便。生产环境直接编译出结果可以使用。

```
<body>
  <script src="localhost:8080/app.js"></script>
</body>
```

以上大致就是开发环境中Vue的关键代码。所以在开发环境中，我们的想法是，利用webpack-dev-server启动Vue部分的代码，hugo部分的代码中判断环境，如果是开发环境就插入`<script src="localhost:8080/app.js"></script>`这段代码。

```
<body>
  <script src=".../vender.vue.[hash].js"></script>
  <script src=".../vendor.[dep].[hash].js"></script>
</body>
```

以上大致就是生产环境中Vue的关键代码。也就是很多的js业务代码，运行时的依赖文件，css文件等，要把这些依赖引入到文件之中。所以在生产环境中，我们的想法是，利用HTML-webpack-plugin把我们所有的依赖的引入代码打包成一个文件，然后在hugo中判断是生产环境就引入并序列化这个文件的内容（hugo 有一个叫 [readFile](https://gohugo.io/functions/readfile/) 的函数可以读取一个文件内容并序列化到模板文件中）。


## 文件结构设计

该项目的文件结构如下

```
root(hugo root folder)
  - content
  - layouts
    - ...(template files)
  - ui(Vue root folder)
  package.json
```

其中，在 template files 中代码的末尾加入如下代码。

```
<div id="root"></div>

{{ if eq (getenv "HUGO_DEV") "true" }}
  <script src="http://localhost:8800/app.js"></script>
{{ else }}
  {{ readFile "/ui/dist/body.partial" | safeHTML }}
{{ end }}
```

这样，在生成的模板文件中，会根据环境来判断引入的script标签。在开发环境使用webpack-dev-server生成的app.js文件。在生产环境中，利用HTML-webpack-plugin生成的文件，把项目需要是引入的文件全部导入。

最后，在ui文件夹下就可以看做一个完整的Vue的项目。配置webpack分环境进行打包。开发环境下起webpack-dev-server，生产环境下打包进dist目录。具体的webpack配置后文将细说。

## 数据获取方式

由于要用Vue来进行网页的编码，那么我们就不可能像使用hugo一样直接在模板文件中写HTML，而是要利用Vue来生成不同的页面。要生成不同的页面，其关键点就在于同类型的页面要获取不同的数据。所以说，我们的做法是预先把数据存储在网页之中。

我们获取的数据源一共有两个，一个是定义在data文件夹中，一个是定义在content文件夹中md文件的frontmatter中。我们可以在模板文件中，将这些数据定义在window下，作为全局变量。这样就可以在Vue中进行调用这些数据。

```
{
  window.data = "{{ Site.Data.config }}"
  window.title = "{{ .Title }}"
}
```

还有一部分数据是定义在md文件中的markdown。这一部分也可以将其挂载在模板文件中，但是使用 `display:none` 进行隐藏。然后再使用Vue操作这一部分的DOM即可。

```
<div id="_page-content" style="display: none">
  {{ .Content | markdownify }}
</div>
```

## 打包方式

我们将打包的命令配置在package.json文件中。具体的命令定义如下。

```
"scripts": {
  "hugoDev": "cross-env HUGO_DEV=true hugo server --bind 0.0.0.0 --noHTTPCache --disableFastRender",
  "webpackDev": "cd ui && node ../node_modules/webpack-dev-server/bin/webpack-dev-server.js",
  "hugoBuild": "hugo -d dist",
  "webpackBuild": "cd ui && rm -rf dist && cross-env BUILD=true node ../node_modules/webpack/bin/webpack.js --prod --progress --hide-modules",
  "assetCopy": "cp -r ui/dist dist/assets",
  "dev": "npm-run-all --parallel hugoDev webpackDev",
  "clean": "rm -rf dist",
  "build": "yarn clean && yarn webpackBuild && yarn hugoBuild && yarn assetCopy"
}
```

开发环境使用的命令是`yarn dev`。我们看到这里使用了`npm-run-all`插件。因为在开发环境webpack-dev-server和hugo的任务并不会结束，所以直接使用&&符号进行串行的任务并不能执行两个命令，所以要利用他让我们可以一个命令同时执行hugo和webpack的命令。我们可以看到，这个命令同时执行了两个命令。第一个 hugoDev 就是在启动开发环境hugo的编译流程，我们可以注意到其中一开始有一段`cross-env HUGO_DEV=true`的代码就是为了给 template files 中的代码区分环境用的。我们回顾一下在上一节中提到，可以发现插入在末尾的代码中判断环境使用的变量就是 `HUGO_DEV`。第二个命令就是启动webpack-dev-server。这样开发环境就可以正常使用了。

生产环境使用的命令是`yarn build`。这个编译流程分为了四步。第一个命令`yarn clean`用于清除上一次编译留下来的文件。第二个命令`yarn webpackBuild`是执行webpack的打包流程，把Vue的运行时代码打包成一个文件夹。第三个命令`yarn hugoBuild`是执行hugo的打包流程。最后一个命令`yarn assetCopy`把webpack打包出的代码复制到hugo打包出的文件夹下，生成一个最终的项目打包代码文件夹。

## webpack 配置

webpack 配置文件中的大部分都与普通的Vue项目没有什么区别，唯一有区别的地方在于 HTML-webpack-plugin 的配置。

首先，在开发环境，并不需要做任何特殊的配置。编译输出了一个app.js的文件给webpack-dev-server，直接就可以使用了。

在生产环境的编译下，对配置做如下修改。

```
new HTMLWebpackPlugin({
  inject: false,
  minify: false,
  filename: "head.partial",
  templateContent: ({htmlWebpackPlugin}) => `${htmlWebpackPlugin.tags.headTags}`,
})

new HTMLWebpackPlugin({
  inject: false,
  minify: false,
  filename: "body.partial",
  templateContent: ({htmlWebpackPlugin}) => `${htmlWebpackPlugin.tags.bodyTags}`,
})
```

加入了上述两个HTML-webpack-plugin的配置后，这两者将分别把JS文件和CSS文件的引入标签代码打包成了 body.partial 和 head.partial 两个文件。让我们来看看这两个文件。

```
// body.partial
<script src="/assets/js/vendor.runtime.bf9a7ce533402d6e1e4b.js"></script><script src="/assets/js/vendor.vue.10426c5f37b9909f3356.chunk.js"></script><script src="/assets/js/vendor.core-js.0dd09ca0b957f339ec22.chunk.js"></script><script src="/assets/js/vendor.normalize.css.c25d28ba90daebf55ddd.chunk.js"></script><script src="/assets/js/vendor.velocity-animate.96372836ed735c252945.chunk.js"></script><script src="/assets/js/app.c1b601945336515e1a70.chunk.js"></script>
```

```
// head.partial
<link href="/assets/css/vendor.normalize.css.24bf1742e3764eb5de3c.chunk.css" rel="stylesheet"><link href="/assets/css/app.fd7d6ea5c66bbd36de53.chunk.css" rel="stylesheet">
```

然后上述的两个文件在生产环境下分别被以下的代码引入模板文件。

```
{{ if not (eq (getenv "HUGO_DEV") "true") }}
  {{ readFile "/ui/dist/head.partial" | safeHTML }}
{{ end }}

{{ if eq (getenv "HUGO_DEV") "true" }}
  <script src="http://localhost:8800/app.js"></script>
{{ else }}
  {{ readFile "/ui/dist/body.partial" | safeHTML }}
{{ end }}
```

这样就完成了一个 Hugo with Vue 项目的配置。
