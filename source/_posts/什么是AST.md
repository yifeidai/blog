---
title: 什么是AST
date: 2020-08-30 23:59:04
tags:
  - AST
categories:
  - js
---

AST 是 Abstract Syntax Tree（抽象语法树）的缩写。简单的来说就是用来解析你写的代码结构的结果。

AST 的作用是非常巨大的。我们平时最常用的 babel，webpack 等等生产力工具都会用到 AST 这个东西。

现在，我们就来简单的看一看一个实际的 AST 的例子吧。

```
function add(a, b) {
  return a + b
}
```

现在我们写了一个简单的 add 函数。希望将他解析成一个 AST。

现在存在很多种解析的引擎

- esprima
- acron
- Traceur
- UglifyJS2
- shift
- ...

我们选择其中的一种进行解析。

```
const esprima = require("esprima")
const code = `
function add(a, b) {
  return a + b
}
`
const ast = esprima.parseScript(code)
console.log(JSON.stringify(ast, null, 2))
```

将以上的代码进行解析，转化为 AST 的结果是

```
{
  "type": "Program",
  "body": [
    {
      "type": "FunctionDeclaration",
      "id": {
        "type": "Identifier",
        "name": "add"
      },
      "params": [
        {
          "type": "Identifier",
          "name": "a"
        },
        {
          "type": "Identifier",
          "name": "b"
        }
      ],
      "body": {
        "type": "BlockStatement",
        "body": [
          {
            "type": "ReturnStatement",
            "argument": {
              "type": "BinaryExpression",
              "operator": "+",
              "left": {
                "type": "Identifier",
                "name": "a"
              },
              "right": {
                "type": "Identifier",
                "name": "b"
              }
            }
          }
        ]
      },
      "generator": false,
      "expression": false,
      "async": false
    }
  ],
  "sourceType": "script"
}
```

我们可以通过上面这个 json 来很好的描述出 add 函数的代码。

在我们的日常的生产力工具中，AST 占据了不小的地位。无论是 uglify，es6 到 es5 的转化，只要是对于我们代码结构解析的都无法离开 AST 的存在。这是因为 AST 不止可以解析代码结构，同样的也可以操作代码结构并生成修改后的代码结构。解析 -> 操作 -> 生成 这三步，正是这些生产力工具的基本原理。
