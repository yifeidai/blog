---
title: git不常用命令整理（1）
categories:
  - git
tags:
  - git
---

本文是对于一些不是很常用的 git 命令的整理，让我们想要用到一些 git 不常用的功能时，心里可以有一点逼数，知道哪些是可以做的，哪些是做不到的。

## 基础的 git 命令

在写不常用的命令前，这里是我整理的一些比较常用的一些 git 命令。如果发现自己不认识的话就自己去谷歌一下补补课吧。

- git init
- git config
- git clone
- git add
- git status
- git diff
- git commit
- git mv
- git rm
- git reset
- git branch
- git checkout
- git merge
- git log
- git stash
- git tag
- git fetch
- git pull
- git push
- git remote
- git cherry-pick
- git rebase
- git revert
- git help

## 不常用的 git 命令

以下是一些不常用的 git 命令。由于这是一篇总结性的文章，所以这里只会简单的说明每个命令的功能，让大家知道 git 可以干些什么。如果想要知道更加详细具体的用法，或者遇到了某些不知道含义的名词，可以去查看 git 文档。

- git archive: 将整个项目打包并压缩成一个文件，**不包含 git 历史**

- git bisect: 在项目遇到 bug 时，用这个命令可以帮助你用二分查找法的方式迅速定位是哪一个 commit 引入了这个 bug

- git bundle: 将整个项目打包成一个文件，**包含 git 历史**

- git citool: 这是一个图形化 `git commit` 命令，是 `git gui citool` 的 alias

- git clean: 删除所有不在暂存区的**文件或目录**

- git describe: 查找从提交可访问的最新标记。 如果标签指向提交，则只显示标签。 否则，它将标记名称与标记对象之上的其他提交数量以及最近提交的缩写对象名称后缀

- git format-patch: 打包出一个包含 commit 信息的 patch 文件。可以只用 `git am` 把打包文件应用于另一个 git 仓库。类似于使用 `git diff` 生成的 patch 文件，只不过多了 commit 信息。

- git gc: gc 是 garbage collection 的缩写，主要是用于优化磁盘空间

- git grep: 用于在 git 历史中进行搜索

- git gui: 展示 git 的 gui 界面

- gitk: 展示 git 图形化操作工具

- git notes: 为提交添加评注

- git range-diff: 展示两组提交之间的 diff。比如我在master上提交了一些 commit，你也在master上提交了一些 commit（由于没有 push，没冲突），对你我提交的内容进行 diff

- git restore: 类似于 `git checkout --`

- git shortlog: 输出所有的git 的 commit msg，以用户进行分组

- git show: 展示一个或者多个对象。如 `git show HEAD` 是展示当前最新提交的 diff 信息

- git sparse-checkout: 只下载远程仓库中某一个特定文件夹

- git submodule: 在一个 git 仓库中允许建立另一个 git 仓库。但是可以保持各自的工作流独立。常用于多个项目复用同一个包。

- git switch: 切换分支，相当于 git checkout

- git worktree: 在存在一个 git 仓库的情况下，在另一个文件夹下创建这个项目某一个分支的代码，但是新的文件夹下的 .git 并不会维护全部的代码历史，只会标明指向原来的项目。这样可以节省磁盘空间。也省了切换分支的开销。常用于大型项目。

- git fast-export: 将提交导出为 git-fast-import 格式

- git fast-import: 其他版本库迁移至Git的通用工具

TODO， 敬请期待下一期

- git filter-branch: 重写 git 历史。极其危险的一个指令，慎用。

- git mergetool: 打开 mergetool（解决冲突的工具），需要自定义

- git pack-refs:

- git repack:

- git replace:

- git annotate:

- git blame: 查看代码某一次改变的是由哪个人做的详细信息。

- git bugreport:

- git count-objects:

- git difftool:

- git fsck:

- git instaweb:

- git merge-tree:

- git rerere:

- git show-branch:

- git verify-commit:

- git verify-tag:

- git whatchanged:

- git archimport:

- git cvsexportcommit:

- git cvsimport:

- git cvsserver:

- git imap-send:

- git p4:

- git quiltimport:

- git request-pull:

- git send-email:

- git svn:

- git apply:

- git checkout-index:

- git commit-graph:

- git commit-tree:

- git hash-object:

- git index-pack:

- git merge-file:

- git merge-index:

- git multi-pack-index:

- git mktag:

- git mktree:

- git pack-objects:

- git prune-packed:

- git read-tree:

- git symbolic-ref:

- git unpack-objects:

- git update-index:

- git update-ref:

- git write-tree:

- git cat-file:

- git cherry:

- git diff-files:

- git diff-index:

- git diff-tree:

- git for-each-ref:

- git get-tar-commit-id:

- git ls-files:

- git ls-remote:

- git ls-tree:

- git merge-base:

- git name-rev:

- git pack-redundant:

- git rev-list:

- git rev-parse:

- git show-index:

- git show-ref:

- git unpack-file:

- git var:

- git verify-pack:

- git daemon:

- git fetch-pack:

- git http-backend:

- git send-pack:

- git update-server-info:

- git http-fetch:

- git http-push:

- git parse-remote:

- git receive-pack:

- git shell:

- git upload-archive:

- git upload-pack:

- git check-attr:

- git check-ignore:

- git check-mailmap:

- git check-ref-format:

- git column:

- git credential:

- git credential-cache:

- git credential-store:

- git fmt-merge-msg:

- git interpret-trailers:

- git mailinfo:

- git mailsplit:

- git merge-one-file:

- git patch-id:

- git sh-i18n:

- git sh-setup:

- git stripspace:

## reference

- [git reference](https://git-scm.com/docs)
