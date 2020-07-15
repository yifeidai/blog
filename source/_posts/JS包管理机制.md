---
title: JS包管理机制
date: 2020-07-15 11:26:16
categories:
  - js
tags:
  - AMD
  - CMD
  - ES6 MODULE
---

## 最初的JS模块化

最初的JS是不存在规范化的模块化管理的，所有的函数和变量会直接污染全局的命名空间。最初人们的解决方法是使用立即执行函数(IIFE: Immediately Invoked Function Expression)进行模块化。

```
// 全局变量
var global = "hello world"
(function() {
  // 只能在闭包内访问的变量
  var a = 1
})()
```

但是仅仅这样是不够的。2009年 Mozilla 工程师 Kevin Dangoor 发起了 ServerJS 项目（后更名为CommonJS），后被我们熟知的 Node.js 使用作为模块管理的规范，也就是后来广为人知的 CommonJS 规范。

## CommonJS

CommonJS 使用入如下语法定义一个模块。 CommonJS 暴露了 `module.exports` 作为了对外界的接口。CommonJS 同时定义了 `exports` 作为了 `module.exports` 的引用。同样可以使用他来作为输出对象。

```
// module.js
const a = 1
function b() {
  console.log("hello world")
}
module.exports = {
  b: b,
}

exports.a = a
```

这时候，如果想在另一个模块引用这些函数以及变量

```
// main.js
const a = require("module").a
const b = require("module").b
```

如果想让模块直接暴露一个函数或变量，可以这么写

```
// module.js
const a = 1
module.exports = a
```

这时候如果想要使用这个变量

```
// main.js
const a = require("module") // 1
```

但是，如果你都一个小机灵说，既然同样可以使用 `exports` 来输出对象，那我这样写怎么样？

```
// module.js
const a = 1
exports = a
```

这样是不行的。因为之前说过， `exports` 是作为 `module.exports` 的引用存在的，你这样写相当于改变了 `exports` 的指向。程序对外暴露的只是 `module.exports`，这时的 `exports` 当然无法生效了。看如下代码你就明白了。

```
const module = {
  exports: {},
}
const exports = module.exports
// 以上代码是程序的预定义部分，将 module.exports 向外暴露
const a = 1
exports = a // 这时 module.exports 仍然为空对象
```

CommonJS 致力于服务端的生态。但是对于浏览器端 CommonJS 是无法接受的。在服务端，在引用一个模块时，从磁盘中读取一个模块的延时是很短的，但是在浏览器端，网络的延迟导致了模块的加载可能会阻碍其他代码的运行。所以 CommonJS 是无法被浏览器端所接受的。这时候， AMD 规范也就应运而生。

## AMD

AMD 规范采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。require.js 是最常见的实现 AMD 规范的库。

如果你想要定义的模块没有依赖

```
define(function() {
  // your module
})
```

如果你想要定义的模块存在依赖，定义的回调函数将在所以依赖加载完，并且初始化完成后运行。

```
// define 第一个参数是依赖列表，分别对应第二个参数回调函数的每一个函数参数
define(["a", "jquery", "c"], function(a, $, c)) {
  $("body").innerHTML = "hello world"
  a.doSomething()
  c.doSomething()
}
```

## CMD

## ES6 MODULE

## reference

[前端模块化：CommonJS,AMD,CMD,ES6](https://juejin.im/post/5aaa37c8f265da23945f365c)
[CommonJS规范](https://javascript.ruanyifeng.com/nodejs/module.html#toc0)
