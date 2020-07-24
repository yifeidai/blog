---
title: JS包管理机制
date: 2020-07-15 11:26:16
categories:
  - js
tags:
  - AMD
  - CMD
  - UMD
  - ES6 MODULE
  - webpack
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

但是，如果你抖一个小机灵说，既然同样可以使用 `exports` 来输出对象，那我这样写怎么样？

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

require.js 的 API 是这样子的

```
define(id?, dependencies?, factory)
```

第一个参数是这个模块的名字。第二个参数是这个模块的依赖。第三个参数是定义这个模块的代码主题。如果你想要定义的模块存在依赖，定义的回调函数将在所有的依赖加载完，并且初始化完成后加载。

```
// define 第一个参数是依赖列表，分别对应第二个参数回调函数的每一个函数参数
define(["a", "jquery", "c"], function(a, $, c)) {
  $("body").innerHTML = "hello world"
  a.doSomething()
  c.doSomething()
}
```

如果你想引用一个模块，第一个参数是依赖列表，第二个参数是所有模块加载完后的回调函数

```
require(["jquery"], function($) {
  $("body").innerHTML = "hello world"
})
```

但是 玉伯 认为 RequireJS 不够完善，于是就催生了 CMD

## CMD

SeaJS 是实现 CMD 规范的一个库。

不同于 AMD 推崇的**依赖前置**, CMD 推崇的是**依赖就近**。他们俩都会提前把依赖下载下载下来，但是不同的是 AMD 在下载后悔立即执行依赖模块的代码，但是 CMD 只有在你实际使用的时候，也就是你调用 `require` 的时候才会执行被你所 `require` 的模块的代码。这两种模式的优劣社区中有很多中看法。仁者见仁，智者见智。

SeaJS 的 API 是这样子的

```
define(function(require, exports, module) {
  var $ = require("jquery.js")
  $("body").innerHTML = "hello world"
});
```

## ES6 MODULE

之前提到的几种模块化标准都是不是一种标准，而是由社区提出并广受大众接受的一种方案。而 ES6 MODULE 则是在语言标准层面实现的模块化。目前所有的现代浏览器都已经支持了这种语法。

```
// 导出早前定义的函数
const myFunction = () => {}
export { myFunction }
// 或者作为默认导出，相当于 export { myFunction as default } 的语法糖
export default myFunction

// 导入模块
import { foo, bar } from "module.js"
// 导入默认模块，相当于 import { default as foo } from "module.js" 的语法糖
import foo from "module.js"
```

这种声明式而非命令式的语法让静态分析成为了可能。在编译时和写代码时带来了极大的益处。

和 CommonJS 不同的是， import 导出的是一个引用，而 require 导出的是一个拷贝。也就是说，在 CommonJS 中如果改变一个模块内部的值是不会对引用他的地方的值造成影响的，但是 ES6 MODULE 是有影响的。举一个简单的例子。

```
// 对于 CommonJS
// module.js
let a = 1
const adder = () => a++
module.export = {
  a,
  adder,
}

// main.js
const { a, adder } = require("module.js")
console.log(a) // 1
adder()
console.log(a) // 1
```

```
// 对于 ES6 MODULE
// module.js
let a = 1
const adder = () => a++
export { a, adder }

// main.js
import { a, adder } from "module.js"
console.log(a) // 1
adder()
console.log(a) // 2
```

## UMD

UMD 其实是一种利用 IIFE(立即执行函数) 来兼容 AMD 和 CommonJS 的一种形式。使你的代码可以在多个不同平台上，不同的模块化标准都可以运行。

```
(function(global, factory) {
  // 判断是 CommonJS 环境
  typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
  // 判断使用 AMD 标准，如果想要兼容 CMD，多加一条判断 define.cmd 即可
  typeof define === 'function' && define.amd ?
    define(factory)
   :
   // 否则就作为全局变量挂在当前环境的 this 上
  (global.NAME = factory());
}(this, (function () {
  // 你自己定义的模块
  return obj;
})));
```

## Webpack 中的模块化打包

那么在现在大家常用的打包工具 Webpack 中，模块化是如何进行编译处理的呢？让我们看一个例子。节选自 [lq782655835/webpack-module-example](https://github.com/lq782655835/webpack-module-example)

```
// src/add.js
export default function(a, b) {
    let { name } = { name: 'hello world,'}
    return name + a + b
}

// src/main.js
import Add from './add'
console.log(Add, Add(1, 2))
```

```
// modules是存放所有模块的数组，数组中每个元素存储{ 模块路径: 模块导出代码函数 }
(function(modules) {
// 模块缓存作用，已加载的模块可以不用再重新读取，提升性能
var installedModules = {};

// 关键函数，加载模块代码
// 形式有点像Node的CommonJS模块，但这里是可跑在浏览器上的es5代码
function __webpack_require__(moduleId) {
  // 缓存检查，有则直接从缓存中取得
  if(installedModules[moduleId]) {
    return installedModules[moduleId].exports;
  }
  // 先创建一个空模块，塞入缓存中
  var module = installedModules[moduleId] = {
    i: moduleId,
    l: false, // 标记是否已经加载
    exports: {} // 初始模块为空
  };

  // 把要加载的模块内容，挂载到module.exports上
  modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
  module.l = true; // 标记为已加载

  // 返回加载的模块，调用方直接调用即可
  return module.exports;
}

// __webpack_require__对象下的r函数
// 在module.exports上定义__esModule为true，表明是一个模块对象
__webpack_require__.r = function(exports) {
  Object.defineProperty(exports, '__esModule', { value: true });
};

// 启动入口模块main.js
return __webpack_require__(__webpack_require__.s = "./src/main.js");
})
({
  // add模块
  "./src/add.js": (function(module, __webpack_exports__, __webpack_require__) {
    // 在module.exports上定义__esModule为true
    __webpack_require__.r(__webpack_exports__);
    // 直接把add模块内容，赋给module.exports.default对象上
    __webpack_exports__["default"] = (function(a, b) {
      let { name } = { name: 'hello world,'}
      return name + a + b
    });
  }),

  // 入口模块
  "./src/main.js": (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__)
    // 拿到add模块的定义
    // _add__WEBPACK_IMPORTED_MODULE_0__ = module.exports，有点类似require
    var _add__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/add.js");
    // add模块内容: _add__WEBPACK_IMPORTED_MODULE_0__["default"]
    console.log(_add__WEBPACK_IMPORTED_MODULE_0__["default"], Object(_add__WEBPACK_IMPORTED_MODULE_0__["default"])(1, 2))
  })
});
```

## reference

[前端模块化：CommonJS,AMD,CMD,ES6](https://juejin.im/post/5aaa37c8f265da23945f365c)
[Webpack 模块打包原理](https://juejin.im/post/5c94a2f36fb9a070fc623df4)
