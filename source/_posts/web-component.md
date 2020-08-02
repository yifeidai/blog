---
title: web component
date: 2020-08-02 20:53:41
tags:
  - Web Component
categories:
  - HTML
---

## Web Component 概况

Web Component 提供了原生的**组件化**支持。也就是说，这时可以在不使用 React, Vue, Angular 等框架的情况下，浏览器原生所提供的组件化功能。最新版本的 Chrome, Safari, Edge 都已经对 Web Component 提供了支持。对于那些不支持的版本，同样也可以引入 polyfill 来解决。引入 polyfill 最多可以兼容到 IE11，也就是说，在大多数现代浏览器中都是可以使用 Web Component 的。

## Web Component 的基本用法

以下的代码都写在 my-button.js 中

```
const template = document.createElement('template');

template.innerHTML = `
  <div>
    <button>my button</button>
  </div>
`
```

如果我们想要定义一个组件，首先我们要定义他的 HTML 模板。

```
class Button extends HTMLElement {
  constructor() {
    super();

    this._shadowRoot = this.attachShadow({ mode: 'open' });
    this._shadowRoot.appendChild(template.content.cloneNode(true));
  }
}
```

其次，我们要定义这个组件的内部行为。我们看到，Web Component 的定义采取了类继承的定义方式。继承了 HTMLElement 类。所以在这个类中我们可以使用一切 Element 有的方法。
在构造函数中，第一行 `attachShadow` 获取到了当前组件的根节点(称为 shadowRoot)。这里设置了 mode 为 open，意思是可以直接通过 JS 访问和修改该元素。如果设置为 closed，那么将无法被 JS 访问及修改。例如 video 标签。
下一行中，把 template 元素克隆了一下加入到了 HTML 中。注意一定要**克隆**节点。因为在多次调用这个组件时，如果不克隆而是直接 appendChild 的话，几个组件就会公用这个节点而引起问题。

```
window.customElements.define('my-button', Button);
```

最后这一行代码申明了，当使用 my-button 标签的时候，使用我们刚刚定义的类 Button 来描述他的行为。请注意，为了区分 Web Component 与平常的 HTML 标签，Web Component 强制规定了在申明的标签名中必须要带有 `-` 符号。

这些代码合在一起，组成了定义一个 Web Component 的文件。

```
<!DOCTYPE html>
<html>
<head>
<script type="text/javascript" src="./my-button.js"></script>
</head>
<body>
  <my-button></my-button>
</body>
</html>
```

最后在想要使用 Web Component 的地方引入这个文件，并像普通的 HTML 标签一样使用即可。

## Web Component 的生命周期与传参

Web Component 提供了 `connectedCallback` 的生命周期。这个生命周期在这个组件被插入 DOM 树后触发。相当于 React 的 componentDidMount， Vue 的 mounted 生命周期。
与之对应的是 `disconnectCallback`，在元素被移出 DOM 数后触发。

```
// index.html
<my-button label="hi"></my-button>

// my-button.js
static get observedAttributes() {
  return ["label"]
}

attributeChangedCallback(attr, oldVal, newVal) {
  switch(attr) {
    case "label":
      document.querySelector("button").innerHTML = newVal
  }
}
```

Web Component 提供了生命周期 `attributeChangedCallback` 来提示属性的变化。但是只有在 `observedAttributes` 中注册过的变量才会触发这个生命周期。这个生命周期在组件初始化时会触发，之后在任一注册过的属性发生变化时也会触发。

```
<my-button></my-button>

const button = document.querySelector("my-button")
button.label = "hi"
```

单纯的这样定义和改变一个组件的属性是不会触发 `attributeChangedCallback` 的生命周期的。我们可以通过使用 getter 和 setter 的方式来控制和获取。

```
class Button extends HTMLElement {
  ...

  set label(value) {
    this.setAttribute("label", value)
  }

  get label() {
    return this.getAtribute("label")
  }

  attributeChangedCallback(attr, oldVal, newVal) {
    this.render()
  }

  render() {
    // render HTML
  }
}
```

## 使用 Web Component 内部的方法

```
// my-button.js
class Button extends HTMLElement {
  ...
  doSomething() {

  }
}

// index.html
<my-button></my-button>

document.querySelector("my-button").doSomething()
```

我们可以直接访问我们定义的 Web Component 下定义的方法。

## Web Component 的事件

```
// my-button.js
connectedCallback() {
  this.$button.addEventListener("click", () => {
    this.$button.dispatchEvent(
      new CustomEvent("test", {
        detail: "hello,
        composed: true,
      })
    );
  })
}

// index.html
document.querySelector("my-button").addEventListener("test", value => {
  console.log("test", value) // CustomEvent Object
})
```

当想要向外触发事件的时候，先使用 `CustomEvent` 自定义一个事件，其中 detail 字段可以是任意值，为向外传递的信息。必须设置 `composed: true`，否则将无法在组件外监听到信息。然后使用 dispatchEvent 在组件的一个 HTML 元素上触发事件。这样就可以在外界监听到这个事件。

## Shadow DOM

你所定义的 Web Component 全部都封装在 Shadow Dom 中。你在其中定义的 CSS 将不会污染外部的 CSS。但是你可以通过外部来改变 Shadow DOM 内部的样式。推荐的方法是在 Shadow DOM 内部使用 CSS 变量，然后在外部定义不同的 CSS 变量值来控制内部的样式。所有的自定义组件的元素将被挂载在一个被称为 Shadow Root 的根元素上。这个元素的 css 选择器为 `:host`。

## Web Component 的 slot

Web Component 中的 slot 和 Vue 中的 slot 形式非常相似。

```
// my-button.js 的 template 定义部分
<div>
  <button>
    <slot></slot>
  </button>
  <slot name="after"></slot>
</div>

// index.html
<my-button>
  <div>hello</div>
  <div slot="after">after</div>
</my-button>

// 实际渲染
<div>
  <button>
    <slot>
      <div>hello</div>
    </slot>
  </button>
  <slot name="after">
    <div>after</div>
  </slot>
</div>
```

有默认的 slot 和具名的 slot，他们分别将插入在各自对应的 slot 的内部。slot 也有他自己的 CSS 选择器： `::slot(after)` 指名字为 after 的 slot 的 CSS 选择器。

## 扩展原生元素

当你想让你的元素继承一个原生元素所有的特性时，你可以这样定义你的组件。

```
class MyButton extends HTMLButtonElement {
  ...
}

customElements.define("my-button", MyButton, { extends: "button" });
```

只需要在继承时继承那个原生组件的类，并且在 define 时的第三个属性中申明 extends 的原生标签的名字。这是就可以使用这个元素。这个元素可以通过 is 属性来使用。

```
<button is="my-button"></button>
```

但是在扩展元素时不能尝试创建 Shadow Root。如果尝试创建 Shadow Root 将会报错。

扩展原生元素的另一个好处是在子元素被限制的时候也可以使用。例如 thead 的子元素必须是 tr。这时如果你想使用一个自己定义的 Web Component 作为子元素的话是不行的，只有使用这种扩展原生元素的方法，写成 `<tr is="my-tr"></tr>` 才能正常使用。
