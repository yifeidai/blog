---
title: webpack5迁移记录
date: 2021-03-01 01:03:19
tags:
  - webpack

categories:
  - js
---

webpack5 已经发布了好久了，但是我们现在项目构建使用的还是 webpack4。也是时候尝试一下使用 webpack5。感受一下 webpack5 有什么新的功能和改进了。

## webpack5 相对于 webpack4 的提升

[提升](https://webpack.js.org/blog/2020-10-10-webpack-5-release/#general-direction)

## 升级准备

webpack 提供了专门的文档，[迁移到webpack5](https://webpack.js.org/migrate/5/)，来从v4到v5进行升级。这是在官方文档中，他们建议的迁移过程。

1. Node.js 至少需要 10.13.0 版本
2. 升级 plugin 和 loader
3. 确认构建没有 error 和 warning
4. 确保设置了 mode 选项
5. [更新一些过时的配置](https://webpack.js.org/migrate/5/#update-outdated-options)，这个链接中展示了完整的过时的配置列表
6. 测试 webpack5 的兼容性
7. 将 webpack 的包更新到5
8. [清理一些配置](https://webpack.js.org/migrate/5/#clean-up-configuration)，这个链接中展示了完整的迁移过程中需要注意修改的配置的列表
9. 尝试构建并解决构建的问题

## 升级实践

看了一遍文档后，就要自己尝试实践一下这个过程，看看会遇到哪些问题了。我就以自己最近正负责的一个项目为例进行尝试，对照上述的步骤进行一一的尝试。

1. Node.js 至少需要 10.13.0 版本

这一步没有任何的问题，只是升级了本地的后别忘了升级构建生产包的服务器的 Node.js 版本。

2. 升级 plugin 和 loader

3. 确认构建没有 error 和 warning

4. 确保设置了 mode 选项

5. 更新一些过时的配置

6. 测试 webpack5 的兼容性

7. 将 webpack 的包更新到5

8. 清理一些配置

9. 尝试构建并解决构建的问题
