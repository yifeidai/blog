---
title: JS 异步陷阱
date: 2020-10-04 22:54:47
tags:
---

今天在写代码的时候 linter 报了一个错，名字是 `require-atomic-updates` 的错误。具体的代码形势如下。

```
let _db = {}
async function a() {
  _db.contract = (await axios.get("/api/contracts/info")).data
}
```

我乍一看，这种基础的代码还能有什么规范错误的吗？最后，我发现，其实 eslint 期望检查出的是这种类型的错误。

```
let a = 0
function setA(value) {
  a += (await axios.get("/api/contracts/info")).data
}
```

这种情况下，JS 会记住a的值，如果在这个异步过程中a的值被其他东西改变了，那么将得到错误的结果。

在 GitHub 上有一个关于这个问题的 issue： [require-atomic-updates false positive](https://github.com/eslint/eslint/issues/11899)
