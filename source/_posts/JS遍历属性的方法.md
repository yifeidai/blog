---
title: JS遍历属性的方法
date: 2020-08-09 23:06:47
tags:
  - js
category:
  - js
---

## 什么是可枚举属性

可枚举是指属性内部可枚举属性没设为 true 的属性。对于通过属性赋值的方式出现的属性来说，默认是可枚举的。对于通过 `Object.defineProperty` 方式定义的属性默认是不可枚举的。js中的原型属性是不可枚举的。

```
const a = {}
Object.defineProperty(a, "b", {
  value: "hello",
  // 是否为可枚举属性
  enumerable: true,
})
```

我们可以看到，使用 `defineProperty` 方法可以显式的控制设置的属性是否是可枚举属性。

可枚举属性和不可枚举属性在控制台上有十分明显的区别。可枚举属性在控制台上打印出来是正常的颜色，但是不可枚举属性打印出来是有一定透明度的。

## 遍历可枚举属性的方法

1. for ... in ...

for in 方法可以遍历对先的每一个可枚举属性，包括了**原型链上的可枚举属性**

```
const a = {}
a.b = "b"
a.__proto__.c = "c"
Object.defineProperty(a, "d", {
  value: "d",
  enumerable: false,
})

for(let key in a) {
  console.log(key) // b, c
}
```

2. Object.keys

Object.keys 可以遍历自身对象的所有可枚举属性，区别于 for in ，他**不能**遍历原型链上的可枚举属性

和它类似的还有 `Object.values` 和 `Object.entries`， 他们可以遍历的属性都遵循相同的规则。至少返回值上， Object.values 返回对象的值的数组， Object.entries 返回对象键值对的数组

```
const a = {}
a.b = "b value"
a.__proto__.c = "c"
Object.defineProperty(a, "d", {
  value: "d",
  enumerable: false,
})

console.log(Object.keys(a)) // ["b"]
console.log(Object.values(a)) // ["b value"]
console.log(Object.entries(a)) // [["b", "b value"]]
```

3. Object.getOwnPropertyNames

Object.getOwnPropertyNames 可以遍历自己对象的所有属性，**包括可枚举和不可枚举属性**。但是不能遍历自己原型链上的属性。

```
const a = {}
a.b = "b"
a.__proto__.c = "c"
Object.defineProperty(a, "d", {
  value: "d",
  enumerable: false,
})

const keys = Object.getOwnPropertyNames(a)
console.log(keys) // "b"
```

4. Object.getOwnPropertySymbols

Object.getOwnPropertySymbols 可以遍历自己对象上所有以 Symbol 类型的可枚举和不可枚举属性。这里单独拿出来是因为 Symbol 类型的属性的表现十分的特殊。

- for in 不可以遍历 Symbol 类型的属性
- Object.keys 不可以遍历 Symbol 类型的属性
- Object.getOwnPropertyNames 不可以遍历 Symbol 类型的属性

```
const a = {}
const b = Symbol("b")
a[b] = "b"

console.log(Object.getOwnPropertySymbols(a)) // [Symbol(b)]
console.log(Object.keys(a)) // []
console.log(Object.getOwnPropertyNames) // []
for(let key in a) {
  console.log(key) // 没有输出
}
```

5. Reflect.ownKeys

Reflect.ownKeys 直白的说，可以理解为

```
Object.getOwnPropertyNames(target).concat(Object.getOwnPropertySymbols(target)
```

也就是说，Reflect.ownKeys 可以遍历**包括 Symbol 类型**的所有课枚举和不可枚举属性

```
const a = {}
a.b = "b"
a.__proto__.c = "c"
Object.defineProperty(a, "d", {
  value: "d",
  enumerable: false,
})
Object.defineProperty(a, Symbol("e"), {
  value: "e",
  enumerable: false,
})

const keys = Reflect.ownKeys(a)
console.log(keys) // ["b", "d", Symbol(e)]
```
