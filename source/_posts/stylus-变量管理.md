---
title: stylus 变量管理
date: 2020-11-29 23:27:47
tags:
  - stylus
  - css
---

这两天刚刚启动一个新项目，正好有一点时间来思考一些项目整体上的东西。其中我就遇到了一个问题。我们在使用 stylus 的时候，会定义一些变量，用这些变量来写css。比如说`$font-base = 14px`。但是，如果我们在 JS 中想要使用这个字体的大小，以前我只会直接在代码里面写一个14。再好一点，在函数中定义一个变量`const FONT_BASE = 14`。或者再开一个专门的文件来定义 JS 中使用的 stylus 的变量，用的时候再引入。

但是，这样的话是违背代码书写的原则的。既然 stylus 的变量定义和 JS 中我们使用的变量是同一个东西，那么，我们当然也希望可以使用一个文件就可以管理 stylus 变量的定义。

经过我的寻找，我发现 stylus 中确实存在这样一个方法来实现这个目标。

那就是 stylus 的 json 函数。具体的文档位置为 [stylus json函数](https://stylus-lang.com/docs/bifs.html#jsonpath-options)

我们可以把想要定义的 stylus 变量都先定义在一个 JSON 文件中，然后我们可以在 stylus 文件中使用 json 函数进行引入。而当我们想在 JS 中使用这些变量，也可以通过这个 JSON 文件引入。这样就实现了同一个文件来管理 stylus 变量的方法。我们看一个例子。

```
// variable.json
{
  "$main": "#234235",

  "$font": {
    "base": "14px",
    "medium": "16px"
  }
}
```

```
// variable.styl
json("./variable.json")
```

这样做相当于在 variable.styl 文件中做了如下定义。

```
$main = #234235
$font-base = "14px"
$font-medium = "16px"
```

我们可以看到，对象的 key 和 value 分别作为了变量名和变量的值。如果有嵌套的多层对象，那么对象将被全部展开到最底层，每一层的 key 将以中划线的形式进行连接。这就是 json 函数的使用方法。

有了这个方法，我们就可以实现之前提到的目标，在同一个文件中管理 stylus 变量在 CSS 和 JS 中的使用。

stylus 还是有很多高级的功能可以使用的，只不过我们平时只是常用他最基础的那些功能而已。在看 json 这个函数的文档的时候，我向下翻了一下，看到了[use 函数](https://stylus-lang.com/docs/bifs.html#usepath)。这个函数的作用是可以自定义一些新的函数，让 stylus 编译器可以识别这些函数。这就给了我们开发者很大的自由度，让我们可以实现一些骚操作。具体的使用方式还是见文档吧。但是这个文档写的不太仔细，详细使用方法说的不是很确切，有时间我可以再研究一下。
