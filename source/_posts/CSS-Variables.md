---
title: CSS Variables
date: 2020-09-07 00:26:40
tags:
  - CSS
  - CSS Variables
categories:
  - CSS
---

## 什么是 CSS Variables

显而易见，这是一个让你可以在 CSS 中使用变量的技术。

## 兼容性

除了 IE 以外，所有主流的浏览器都对 CSS Variables 有很好的支持。但是你可以引入 polyfill，`css-vars-ponyfill` 将让你可以在 IE9+ 的浏览器中使用 CSS Variables。

## 语法

CSS Variables 的语法非常简单。定义变量一定以双中划线开头，使用 var 函数来使用定义过的变量。var 函数的第二个参数支持默认值。当那个变量没有定义的时候将使用默认值。var 函数将第一个逗号后的所有参数视为默认值，不会处理其中的逗号，引号等等。

```
// define
--gray: #F5F5F5
// use
color: var(--gray)
// use with default value
color: var(--gray, #E0E0E0)
border: var(--border, 1px solid gray)
```

如果定义的是一个数字，后面不可以直接跟单位，但是你可以使用 calc 函数来做这件事

```
--twenty: 20

// wrong
height: var(--twenty)px
// right
height: calc(var(--twenty) * 1px)
```

## 作用域

CSS Variables 和正常的 CSS 一样也有作用域的规则。权重更高的变量将生效。一般情况下，我们将CSS Variables 定义在 :root 的 CSS 选择器中。这样所有的地方都可以使用这些变量。

## JS 与 CSS 交互

我们可以使用 JS 设置一些 CSS Variables，也可以通过 JS 读取 CSS Variables。

```
// set
document.body.style.setProperty("--gray", "#E0E0E0")
// get
document.body.style.getPropertyValue("--gray")
// delete
document.body.style.removeProperty("--gray")
```

## 作用

### 定制主题

我们有一个常见的需求，就是网页除了正常的样式外，还要定制一个暗黑模式。这种情况下，CSS Variables 就是一种实现的方案。我们只需要定义两套不同的 CSS Variables，分别定义在暗黑模式和正常模式下的调色板是什么。通过程序控制加载什么变量即可。

### CSS 存储信息

我们可以看到，我们现在可以通过 JS 来读取 CSS Variables，所以我们可以在 CSS 中存储一些信息然后通过 JS 读取。

### JS 设置 CSS

比如说我们要实现一个进度条。那么我们可以通过使用 CSS Variables 的方案，设置他的宽度为一个CSS Variable，然后通过用 JS 改变 CSS Variable 的值来改变进度条的进度。
