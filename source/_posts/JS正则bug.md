---
title: JS正则bug
date: 2020-10-11 23:13:01
tags:
  - js
  - 正则
categories:
  - js
---

让我们来看一段代码

```
const reg = /\d/g
console.log(reg.test("1")) // true
console.log(reg.test("1")) // false
console.log(reg.test("1")) // true
console.log(reg.test("1")) // false
```

这时我们会发现，有两次正则的 test 方法得出的结果是错误的。这是什么原因呢？让我们再看几段代码。

```
const reg = /\d/
console.log(reg.test("1")) // true
console.log(reg.test("1")) // true
```

```
console.log(/\d/g.test("1")) // true
console.log(/\d/g.test("1")) // true
```

我们可以看到，我们换了两种写法，这个 bug 就不再出现了。一个是去掉了正则的 g 这个 flag。一个是不再使用定义的正则常量，而是每次都使用新的正则实例。显然，这个 bug 出现的原因就在与这个正则常量以及 g 这个 flag 身上。让我们再看一段代码。

```
const reg = /\d/g
console.log(reg.lastIndex) // 0
console.log(reg.test("1")) // true
console.log(reg.lastIndex) // 1
console.log(reg.test("1")) // false
console.log(reg.lastIndex) // 0
console.log(reg.test("1")) // true
console.log(reg.lastIndex) // 1
console.log(reg.test("1")) // false
console.log(reg.lastIndex) // 0
```

我们可以看到，这个正则常量有一个叫做 lastIndex 的属性，这个属性是在正则使用了 g 这个 flag 的时候才会有作用。用处是记录上一次正则匹配的结果的索引值 +1，匹配失败则返回0。这样存在 g 这个 flag 的正则可以进行多次匹配。让我们来看一段代码来体会他的作用。

```
const reg = /\d/g
console.log(reg.exec("12")) // ["1", index: 0, input: "12", groups: undefined]
console.log(reg.exec("12")) // ["2", index: 1, input: "12", groups: undefined]
console.log(reg.exec("12")) // null
```

在使用正则进行多次匹配的时候，就是 lastIndex 这个属性记录的上一次匹配的位置，并告诉你他下次开始匹配的位置的索引值。我们也可以修改 lastIndex 来改变。

```
const reg = /\d/g
console.log(reg.exec("12")) // ["1", index: 0, input: "12", groups: undefined]
reg.lastIndex = 0
console.log(reg.exec("12")) // ["1", index: 0, input: "12", groups: undefined]
```

可以看到，这时候又从头开始匹配了。在我们平时的业务代码中，有时候经常会需要一个类型的正则，就比如说检查输入的数字是否是合法的金额，并且在好几个地方都会用到这个正则。这种情况下我们可能就会定义一个常量来储存这个通用的正则。所以我们平时要小心这个问题。

但是，在我们的第一段代码中，显然这种结果不是我们所预期的。这显然是可以称为一个 bug 的。想要避开这个问题，一共有三种解决方法。

```
/\d/g.test("1")
```

第一种我认为是最好的，可以最优雅的解决这个问题。就是每次用正则的时候都使用新的实例，不要定义常量。

```
cosnt reg = /\d/
reg.test("1")
```

第二种就是不使用 g 这个 flag 就不会触发 lastIndex 的问题。但是有时候我们确实会需要 g 这个 flag，并不一定能完美的解决问题。

```
const reg = /\d/g
reg.test("1")
reg.lastIndex = 0
```

第三种就是每次使用过之后，都将 lastIndex 属性重置为 0。这个方法就不是非常优雅。
