---
title: 几种CSS管理方案
date: 2020-08-23 23:04:40
tags:
  - CSS
  - CSS in JS
  - CSS module
  - BEM
categories:
  - CSS
---

CSS 的管理一直都是一个让人头大的难题。在 SPA 中，由于 CSS 的作用域是全局的，所以每一个页面的文件的样式都可能会影响到其他页面的样式。当你无意识中写了一个和其他页面同名的样式时，这无疑是致命的。而当你在对 DOM 结构进行删改时，非常容易遗留下无用的 CSS。所以如何合理的维护一个工程的 CSS 无疑是一个巨大的难题。

## BEM

BEM 是最简单的解决 CSS 命名冲突的方案。是一种通过约定来解决命名冲突的实现方式。

```
.block {}
.block__element {}
.block__another-element {}
.block--modifier {}
```

- `block` 代表了一个命名空间。可能是一个页面，可能是一个组件，也可能是一个组件中的一部分。可以自行约定。
- 所有在 `block` 命名空间下的样式都已 `block` 开头，并已双下划线连接自己的名字
- 中划线作为连字符
- 双中划线连接的代表不同的状态
- 不可嵌套过深的层级，避免出现过长的 CSS

当然这些都是一些约定，所以自然你也可以加入一些自己的约定。

BEM 结构清晰，易于扩展，但是很容易出现 CSS 类名过长，CSS 文件过于繁琐的问题。

## CSS in JS

显而易见的，CSS in JSS 就是在 JS 代码中写 CSS。具体是什么样子的，我们就来看一个最流行的实现 CSS in JS 的库的代码。

```
import React, { Component } from "react"
import styled from "styled-components"

const Wrapper = styled.div`
  background: black
`
const Title = styled.h1`
  background: white
`
class App extends Component {
  render() {
    return (
      <Wrapper>
        <Title>Hello world!</Title>
      </Wrapper>
    )
  }
}

export default App
```

这一份代码应该十分的浅显易懂。每一个标签都由 styled 使用 ES6 的新语法，变迁模板字符串生成一个 HOC。这样每一层样式都定义成了一个组件来进行使用。

他的实现方式并不是简单的使用内联样式，而是生成了一个随机的字符串作为类的名字，并且保证了每一个类名都不会重复。这样就完美的解决了之前提到的样式名称重复的问题。

优点：

- 可以在 CSS 中使用 JS 的变量
- 可以在 CSS 中使用 JS 来进行计算
- 解决了全局命名冲突的问题
- 对于推崇 everything in JS 理念的人很友好

缺点

- 不能使用 post CSS
- 不能使用 CSS 预处理器
- 难以调试，在 devtool 中只能修改当前的组件实例
- 性能不可避免的更低
- 生成的代码有很多冗余
- 将样式和 JS 代码混合在一起，对一些人来说不可接受

除此之外，前面提到的优点都不是没有解决方法的，比如使用 BEM，使用 less，sass 等等。

## CSS Module

CSS Module 的开启方法是只要在 webpack 的 css-loader 配置中的 options 选项中把 module 设为 true 即为开启了 Css Module。

让我们看看 CSS Module 的使用方式。

```
// index.js
import Index from './index.css'

const html = `<div class=${ Index.header }>
	<h1 class=${ Index.title }>CSS Modules</h1>
  <h1 class="title">global class title</h1>
</div>`
```

```
// index.css
.header {
  background: black;
}

.title {
  color: white;
}

:global(.title) {
  composes: header;
  color: green;
}
```

CSS Module 的作用相当于把每一份 CSS 文件的进行了隔离。然后可以 import 进来引用他的名字。当不想只有页面的作用于的类在前面加上 global 即可。这样可以直接在使用他本身的类名而不用 使用 import 进来的名字。可以使用 composes 来复用已存在的类。

他的原理十分的简单，就是 webpack 在编译的时候用哈希的方式另外生成了一套类名。而 import 进来的就是原来的类名和编译后的类名的映射的对象。
