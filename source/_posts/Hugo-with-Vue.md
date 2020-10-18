---
title: Hugo with Vue
date: 2020-09-18 14:31:42
tags:
  - Hugo
  - Vue

categories:
  - HTML
---

本文主要讲述如何在 Hugo 中使用 Vue 进行开发一些复杂交互的网站。

## Hugo

Hugo 是一个 static site generator(静态网站生成)的框架。静态网站生成指的是利用一些模板文件，预先将所有的HTML文件生成好，挂载在服务端的一种技术。利用 Hugo 你就可以生成一系列网页。但是由于这种框架都是只基于简单的 HTML，JS，CSS，而不涉及任何其他的框架，所以如果要开发一些比较复杂的网站的时候就会变得比较困难。这时候，利用一些现代的框架（如Vue，React，Angular等等）来简化我们的开发。这里我们就以 Vue 为例来看看怎么使用 Vue 来配合 Hugo 进行开发。

## 前提知识

本文将假设读者已经了解以下知识：

- Hugo 的基本知识（文件夹结构，编译流程等等）
- Vue 的基本知识
- webpack 配置基础
