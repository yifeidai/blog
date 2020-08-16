---
title: .git 文件夹探秘
date: 2020-08-16 23:26:18
tags:
  - git
categories:
  - git
---

## 什么是 .git 文件夹

.git 文件夹是 git 存储整个仓库的信息的位置。如果想要知道整个仓库的信息，那么只要这个文件夹就足够了。
这个文件夹的结构如下所示。

```
文件夹
- hooks
- info
- logs
- refs
- objects
文件
- config
- description
- FETCH_HEAD
- HEAD
- index
- ORIG_HEAD
- packed-refs
```

接下来我们来一个个看看这些文件夹都是用来干什么的。理解了这些文件夹是干什么的之后想必你对于 git 的理解将会加深一个层次。

在解释 这个文件夹结构之前，我们首先要知道一个基础的知识。那就是 git 和以前的 SVN 这种版本控制系统的差别在哪里。 SVN 这种版本控制系统的原理是每一个改变都会作为一个 diff 记录下来。当前的版本就是所有 diff 之和。而 git 的存储方式是每一个文件的每一个版本都会作为一个快照保存下来。一次 commit 就相当于记录了所有文件的快照。

## hooks

hooks 文件夹存储的是一些脚本，可以在不同的阶段运行，比如 commit、pull 等等。

## info

info 文件夹中有一个文件名字叫 exclude。这个文件是派什么用的呢？顾名思义，其实这个文件的作用和 .gitignore 的作用是一样的，不过这个文件每一个人是可以不一样的，也就是说一个团队的每个人可以有一份自己的 .gitignore 文件。

## logs

顾名思义，这个文件夹的作用就是日志，给你反悔用的。我们看看这个文件夹下的文件结构

```
文件夹
- refs
文件
- HEAD
```

我们打开 HEAD 文件看一下，可以发现是如下的信息。

```
......
9bb7f56e59fb83ea7b7c55a23409f8a2ab64a033 25c9b1108072c99c0643bad5d49092861f14e492 yifeidai <floatdai@gmail.com> 1581662743 +0800	commit: BBS 3114
25c9b1108072c99c0643bad5d49092861f14e492 ce476df4e0c091b1ee00c0009778ca0fa8871c1f yifeidai <floatdai@gmail.com> 1581663490 +0800	commit: BBS 3120
ce476df4e0c091b1ee00c0009778ca0fa8871c1f 1f1295bda02bb5d7e19b66bf3c86264bfdc0ce62 yifeidai <floatdai@gmail.com> 1582288127 +0800	commit: Finish sprint 20
1f1295bda02bb5d7e19b66bf3c86264bfdc0ce62 a6d8378f0fcf66c6f00b61f5ce7150d645727c28 yifeidai <floatdai@gmail.com> 1588214020 +0800	pull: Fast-forward
a6d8378f0fcf66c6f00b61f5ce7150d645727c28 978be520018e6a72359148afe2e5af9f5ba893a4 yifeidai <floatdai@gmail.com> 1588227237 +0800	commit: Add export data in welfare orders
978be520018e6a72359148afe2e5af9f5ba893a4 c1348b61a2dd28ce03a98e1327909b5dc0ba377d yifeidai <floatdai@gmail.com> 1588229105 +0800	commit: Add import data in welfare orders
c1348b61a2dd28ce03a98e1327909b5dc0ba377d 572ecb8ccd0b4259cd2acabbec57445b682d74dd yifeidai <floatdai@gmail.com> 1588242537 +0800	commit: Add edit in welfare goods
572ecb8ccd0b4259cd2acabbec57445b682d74dd 401b1dff157a06b4dc53cf281bfbdb1c231128ae yifeidai <floatdai@gmail.com> 1588245717 +0800	commit: Fix welfare permission bug
401b1dff157a06b4dc53cf281bfbdb1c231128ae 462579259947c59c562df4c43152ef4440e2d3e7 yifeidai <floatdai@gmail.com> 1589969280 +0800	pull: Fast-forward
462579259947c59c562df4c43152ef4440e2d3e7 48b2c8aa20317ba0ee997897cd4b4c3a5ce47460 yifeidai <floatdai@gmail.com> 1589969439 +0800	commit: Change auth of management/userinfo
```

可以看到，这是由一个个 commit 的信息组成的。然后再看看文件的名字： HEAD，显然这个就是 git 的 HEAD 变更的日志呀。再看看 refs 文件夹下的内容。

```
- heads
  - master
- remotes
  - origin
    - HEAD
    - master
    - release
```

显然这个就是一个本地和远程的每一个分支的文件组成结构。打开后可以发现他们的内容和之前的 HEAD 文件是相同的类型。也就是说他们分别记录了本地和远程每一个分支的变动信息。

记得如果我们有时候如果用了 hard 的 git reset 命令时想要找回被自己删掉的代码该怎么办吗？就是使用 `git reflog` 命令，会展示出所有的提交的信息。这些信息的来源就是这个文件夹。

## refs 和 objects

refs 文件夹和 objects 文件夹必须放在一起说明。我们之前说的那些文件夹其实都没有涉及到 git 的核心，也就是 git 的版本控制是怎么实现的。这其中的秘密就在这两个文件夹里面。

首先看一下 refs 文件夹下的文件结构

```
- heads
  - master
- remotes
  - origin
    - HEAD
- tags
  - v1.0.0
  - v1.0.1
  ...
```

和上面 logs 文件夹的文件结构类似，这个文件结构包含了本地和远程所有的分支信息，除此之外还加上了 tags 的信息。那么显然每一个文件都是用来记载一个分支或者 tag 的详细信息的。那么这些文件是怎么记录这些信息的呢？让我们打开其中一个文件来看看。

```
heads/master 文件
48b2c8aa20317ba0ee997897cd4b4c3a5ce47460
```

这个是什么？这个显然是一个 git 的 commit 的 SHA-1 啊。我们知道 git 的每一个提交都会生成一个 SHA-1 来作为这个 commit 的唯一标记。所以我们可以知道 refs 文件夹的作用是保存了每个分支和 tag 的当前的 commit 是什么的信息。但是这个文件夹并没有保存每个 commit 具体内容是什么，也没有保存整条分支，而是只知道一个分支的最新的提交是什么。那么这个任务也就是有 objects 文件夹来完成的。那么久让我们看看 objects 文件夹中有哪些东西。

```
- 0a
- 0b
- 0c
- 0d
- 0e
...
- d0
- d4
- d5
- d6
...
- fc
- fd
- fe
- info
- pack
```

中间省略号部分省略了很多两个字母的文件夹。哇，这些乱七八糟的文件夹到底是干什么的？很显然，这些都是两位的十六进制数组成的。再联想到我们之前 refs 文件夹中记录的内容 SHA-1，显然这是在为 SHA-1 做索引啊。我们打开 0a 这个文件夹，发现里面有两个文件。

```
9bbb2c1faa43e0dcb27e93ec9263ec4a5cc539
f23568e3ccab0fcfd4ce5e561076660e674d4c
```

把这两个文件的前面加上 0a，可以发现这就是两个 SHA-1。也就说，之前在 refs 文件夹中记录的信息的意义找到了。这个 SHA-1 将这个 commit 的信息保存在了 objects 文件夹的一个个文件之中（注意，并不是所有的在 objects 文件夹中的文件都是这个，接下来我会意义说明）。那么就让我们打开之前之前记录在 heads/master 文件中的那串 SHA-1 的文件。由于 git 会对 objects 中的每一个文件做压缩，所以直接打开文件看到的都是乱码。 git 提供了指令 `git cat-file` 来查看 objects 文件夹中每一个文件的内容。那么就让我们输入 `git cat-file -p 48b2c8aa20317ba0ee997897cd4b4c3a5ce47460` 来看看这个文件中到底记录了什么。

```
tree bde4c5ed97de8cf3a311e8edc93ec2c1bbe7224e
parent 462579259947c59c562df4c43152ef4440e2d3e7
author yifeidai <floatdai@gmail.com> 1589969439 +0800
committer yifeidai <floatdai@gmail.com> 1589969439 +0800
```

我们先跳过第一行，看第二行。parent 代表的是这个 commit 的上一个 commit 的内容是什么，这个 commit 的对应的文件同样也可以在 objects 文件夹中找到。也就是说每一个 commit 的具体信息中都记录了上一个 commit 到底是什么。所以说之前的疑惑就得到了解决。在 refs 文件夹中的没有个分支文件都只记录了当前分支最新的 commit 是什么，那么怎么才能得出整个分支是什么样子的呢。那就是靠每一个 commit 都记录了他上一个 commit 是什么。一个个串起来就得到了一整条分支是有哪些 commit 构成的。

接下来看第三第四行，他们分别具体记录了作者和提交者的信息和提交的具体时间。

最后我们来看最重要的第一行。之前的那些信息让我们可以理解了 git 的 commit 树的信息是怎么存储的。但是，具体每一个 commit 他所包含的文件内容是什么还是没有。这个信息就是这个 tree 对象所存储的。tree 的意思其实就是相当于一个文件夹，最顶层的文件夹的信息当然就是一个提交的目录信息了。我们看看 tree 右边是什么？又是 SHA-1，说明这个 tree 的具体信息也是作为一个文件存储在 objects 文件夹之中的。说到这里大家可能有点蒙，objects 文件夹中村的不是所有 commit 的 SHA-1 吗？怎么突然有蹦出来一个具体 commit 的 tree 的 SHA-1？其实他们都是存储在 objects 文件夹之中的。这个文件夹存储了很多信息。可以看成一个索引的系统。那么就让我们看看这个 tree 中的信息是什么？同样的，我们使用 `git log -p bde4c5ed97de8cf3a311e8edc93ec2c1bbe7224e` 来查看这个文件

```
100644 blob cd68a1b91fea8d5f57f6bafe03b81bc0c89a78b0    .browserslistrc
100644 blob 107ed74e6bfd14740b6334a931034b99c542cb5f    .gitignore
040000 tree 0274a81f8ee9ca3669295dc40f510bd2021d0043    .vscode
100644 blob 36109a8ca1e99df196b62750beffd2fb95ffd1a5    README.md
100644 blob 12da593be7f40fed6e53780801e3ad03d54affa4    babel.config.js
040000 tree d810c104ef132cd9765c070e9ee0f9a32db2d309    build
100644 blob a4dbc245f2bc6c829ea3d7ebc1b9d25f5d664cd7    index.html
100644 blob d9745d0b909f1a2497696f50bfd1347f399ca32b    package.json
040000 tree 7c94cf2b4494855cd0b4f92d4a85cbfe36bf8e6b    src
040000 tree 827cbc09e4a73bafe37c80eefd43f53f936cc89e    static
100644 blob cead9117d91f1abc3339be2dcb0f7b71e70ce633    yarn.lock
```

我们可以看到，这个文件中的第二列有两种值，blob 和 tree，之前说过了，tree 就相当于一个文件夹，那么 blob 显然就相当于一个具体的文件。我们再看看最后一列，他们就是每一个文件或文件夹的名字。而第一列则记录了这个文件或文件夹的具体类型等信息。和 linux 文件系统是不是很像。而第三列显然就是这些文件或文件夹的 SHA-1，他们具体的详细信息也将保存在另一个在 objects 文件夹中的文件中。tree 的文件保存的信息都和这个文件差不多，那么让我们看看 blob 类型的文件是什么样子的。同样的我们利用 `git cat-file -p cd68a1b91fea8d5f57f6bafe03b81bc0c89a78b0` 命令来看看 .browserslistrc 文件的内容是怎么保存的。

```
last 2 Chrome versions
```

这个就是这个文件中的具体内容了。

这样一来整个 git 的存储逻辑就非常清晰了。首先在 refs 文件夹中对每一个分支都简建立一个具体的文件。这个文件中保存了每个分支最新的 commit 的信息。接下来每一个 commit 的信息都记录在 objects 文件夹中。首先，每一个 commit 的文件中记录的是提交者的信息。其次是该 commit 的上一个 commit 的信息，这让每一个 commit 都连了起来，记录一个 commit 就相当于记录了整个分支的 commit 信息。最后是 tree，也就是一个文件夹的信息。每一个文件夹的信息文件中都记录了下一层文件夹中的文件和文件夹信息。这就构成了每一次提交的文件结构快照。最后 tree 的末端，也就是文件夹的末端就是 blob 类型，一个个具体的文件。每一个文件的信息也作为一个个单独的快照文件保存在了 objects 文件夹之中。这结合起来就构成了每一个提交的具体内容快照。

## config

显然，这个文件夹就是我们每一个项目自己的 config， `git config --local` 命令中具体的信息就存储在这个文件中

## description

这是一个被 git web 所使用的文件，展示了这个仓库的描述。

## FETCH_HEAD

这个文件保存了 FETCH_HEAD 的引用是什么

## HEAD

打开这个文件，内容如下

```
ref: refs/heads/master
```

这个文件显然记录了本地的 head 的具体信息。

## index

顾名思义，这个文件保存了 git 的 index 区（暂存区）的信息。这个文件也是经过加密的。我们可以通过 `xxd -b -c 4 .git/index` 命令看看这个文件的全貌。

```
00000000: 01000100 01001001 01010010 01000011  DIRC
00000004: 00000000 00000000 00000000 00000010  ....
00000008: 00000000 00000000 00000000 00000001  ....
0000000c: 01011110 01011011 11000111 10010001  ^[..
00000010: 00101111 00001011 00100011 00101010  /.#*
00000014: 01011110 01011011 11000111 10010001  ^[..
00000018: 00100111 00101111 10101000 11100000  '/..
......
```

我们也可以通过 `git ls-files --stage` 来查看仓库中的每一个文件及其对应的文件对象。

```
100644 063b0e4ce79bbd23403f7e8ebfb71fb7779f869a 0       .gitignore
100644 3a814e386167aeb4e5bdc1ddc146612c4a42c154 0       package.json
......
```

具体这个文件存储信息的规则是什么样子的由于非常复杂，可以单独展开作为一篇文章讲，这里就不展开说明了。具体的文档可以参考[git index-format](https://github.com/git/git/blob/master/Documentation/technical/index-format.txt#L63)。

## ORIG_HEAD

这个文件记录了上一次 HEAD 所在位置的信息。也就是用来反悔用的。

## packed-refs

git 会定期执行一个叫做 `git gc` 的命令，gc 是 garbage collection 的缩写。这个命令会将一些暂时用不到的 commit 和分支的具体内容打包起来，打包在 objects 文件夹下的 pack 文件夹下，用来压缩所占用的体积。这个文件就是用来记录这些信息的。
