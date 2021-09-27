---
title: yarn
date: 2021-09-06 14:13:27
tags:
---

## 历史

太古时期，npm 无道，众 JS 开发者苦 npm 久矣。npm 承平已久，不思进取。

这里就历数一下 npm 当年的几大原罪

- 速度慢
- 解析出来的依赖列表不稳定
- 安全性差
- 树状的安装结构
- 命令行输出极其难懂
- ...

故开源社区揭竿而起。其中的佼佼者就是 facebook 的 yarn。

## yarn

针对上述的缺点，yarn 进行了一系列的改进

- 使用 yarn.lock 文件稳定构建的版本
- 每次安装依赖使用 checksum 检查
- 使用扁平的安装结构
- 支持 workspace
- 易懂的命令行输出
- ...

当然以上这些优化在 yarn 的鞭策之下，npm 也同样进行了改进

- 使用 package-lock.json 文件稳定构建的版本
- 每次安装依赖使用 SHA-512 检查
- ...

不过，yarn 还是有一个相对于 npm 无可比拟的优势。yarn 的安装是并行的，而 npm 的安装是串行的。也就是说 npm 在安装完一个包后才能安装下一个包，而 yarn 可以同时安装多个包。


## 一些有用的 yarn 的小知识

### yarn why

可以知道具体某一个包为何被安装

### yarn upgrade-interactive

使用可视化的界面来选择所有的依赖是否升级

### yarn link

用于本地包的开发，将本地的一个文件夹作为一个包，然后可以被另一个项目进行使用

### yarn bin

可执行文件安装的位置，yarn 的 bin 目录经常不在 PATH 中被设置，所以我们可以在 .bashrc 文件中加入 `PATH="$PATH:(yarn global bin)"` 来使通过 yarn 全局安装的文件可以被直接执行

### 全局的缓存文件夹

yarn 的文件下载后是先会被存在一个缓存的文件夹下，然后再拷贝进入 node_modules 目录，这个文件夹可以通过 `yarn cache dir` 命令找到。当一个文件已经被安装过，再次通过 `yarn` 这个命令进行安装，那么将不会通过网络下载这个包，而是直接从缓存中拷贝进入 node_modules 目录

### offline mode

有时候，我们如果不希望通过网络下载包，比如在弱网环境，或者由于公司原因的网络隔离等等。我们就可以开启 offline mode

在当前项目目录下新建 `.yarnrc` 文件

```
yarn-offline-mirror "./npm-packages-offline-cache"
yarn-offline-mirror-pruning true
```

这样我们就开启了 offline mode

这时候，如果我们运行 `yarn` 命令，我们会发现在 npm-packages-offline-cache 目录下多了和全局缓存文件夹下一样的缓存文件
这时候我们再删除 node_modules 文件夹，断开网络，然后再执行 yarn 命令，我们仍然可以正常运行

## yarn2

随着越来越多的新功能的加入，yarn 原来的架构无法满足新需求，所以 yarn 推出了 v2 版本。他们做出了以下的变动

- 从 flow 迁移到 ts 进行开发
- 使用基于插件和模块化的代码架构，让开发者不用理解 yarn 的核心代码就可以通过插件的方式为 yarn 增加新功能

他也引入了不少新的功能和改进

### 更友好的输出信息

![](https://floatdai-blog.oss-cn-beijing.aliyuncs.com/yarn%E5%91%BD%E4%BB%A4%E6%88%AA%E5%9B%BE.png)

我们可以看到，每行日志开头都有了错误码，我们可以快速的通过 [错误码文档](https://next.yarnpkg.com/advanced/error-codes) 定位到问题。
对于每个包的操作关联到了一起，同时还各自高亮了包的名字和版本号

### 更好的 workspace 支持

yarn2 将 workspace 变成了一等公民，这样可以更好的支持 Monorepository。也就是一个项目空间中包含多个小的不同的包。比如 babel 就是一个例子。

他提供了以下便利的功能

- yarn add interactive mode，使用`yarn add xxx -i`，其他 workspace 可能已经使用了这个包的某个版本，可以让用户选择是否复用
- `yarn up`： 一次更新所有 workspace 下的版本，同样也具有 interactive mode
- 更好的发布流程体验
- `yarn workspaces foreach`： 所有 workspace 运行同一个命令
- [contraints](https://next.yarnpkg.com/features/constraints)，给所有 workspace 增加一些约束规则，这样可以通过命令来检查 workspace 是否违反了这些规则
- 像搜索数据库一样查询workspaces的依赖信息，可以使用 `yarn constraints query` 命令查询 workspace 用到的依赖信息

### yarn dlx

`yarn dlx` 是 (download and execute) 的简称，他相当于 npx 的功能，将下载一个脚本并运行这个脚本。由于有缓存的原因，当执行过一次 `yarn dlx` 后再次执行会使用缓存的文件，速度会更快。

### Normalized shell

yarn2 对 Windows 开发环境做了更好的兼容。我们经常会遇到这样一个问题。一个脚本在 Mac 或 linux 环境中可以正常运行，但是在 Windows 上却无法正常运行。这也是我将开发环境迁移到 wsl 的原因。但是 yarn2 自带了一个简单的 shell 解析器，覆盖了90%常用的shell脚本写法，可以让脚本在 Windows 和其他系统中一样正常运行。

### [Protocols](https://next.yarnpkg.com/features/protocols)

在 package.json 中 dependencies 和 devDependencies 字段中，我们可以定义我们的依赖，其中 key 是包名，而 value 指就涉及到了如何解析这个包。protocols 就是约定这个 value 值的语义是如何的

| 名字 | 例子 | 描述 |
|------|------|------|
| Semver | ^1.2.3	| Resolves from the default registry |
| Tag | latest	| Resolves from the default registry |
| Npm alias |	npm:name@... |	Resolves from the npm registry |
| Git | git@github.com:foo/bar.git |	Downloads a public package from a Git repository |
| GitHub | github:foo/bar	| Downloads a public package from GitHub |
| GitHub | foo/bar	| Alias for the github: protocol |
| File | file:./my-package	| Copies the target location into the cache |
| Link | link:./my-folder |	Creates a link to the ./my-folder folder (ignore dependencies) |
| Patch | patch:left-pad@1.0.0#./my-patch.patch |	Creates a patched copy of the original package |
| Portal | portal:./my-folder	| Creates a link to the ./my-folder folder (follow dependencies) |
| Workspace | workspace:* |	Creates a link to a package in another workspace |
| Exec | exec:./my-generator-package |	Experimental & Plugin. Instructs Yarn to execute the specified Node script and use its output as package content |

link 和 portal 的区别？

他们都是指向本地文件系统中的一个 symlink。但是当那个指向的包中包含 package.json 文件的时候，yarn 会解析这个包中的 transitive dependencies（也就是依赖的依赖）

### Zero-Installs

这是 yarn2 带来的一种新的理念。我们平时都会使用 `yarn` 安装，但是 Zero-Install 是指当你从 git 拉取别人的代码后，你无需运行 `yarn` 命令即可立即运行。

在开发的时候，我们经常会碰到一个问题，拉取代码后由于别人引入了一个新的依赖就无法运行了。这就可以解决这个问题

yarn2 认为，根据墨菲定律，任何可能出问题的东西最终都可能会出问题。比如 yarn 出了 bug，网络环境有问题等等。所以为了防止这个，我们就最好运行最少量的代码。

### Zip loading

我们想要实现 Zero-Installs，就必然会面临一个问题一个问题，node_modules 中有海量的文件，巨量的文件让我们不可能把这些文件推送的远程。所以 yarn2 把每个包都各自压缩成了一个 .zip 的文件，PnP 的 runtime(.pnp.cjs) patch 了 fs 模块，让我们可以访问 Zip 文件中打包的文件。

## PnP(Plug'n'Play)

这个功能我觉得是 yarn2 主推的一个新功能，也是和我们日常开发最息息相关，提升体验巨大的一个功能。所以我单独分开来讲他。就是通过 PnP 实现了之前提到的理念 Zero-Installs。当然如果你不同意这个理念，你也可以通过一些设置关闭它。总得来说，他大幅度的提升了打包的速度。

### node_modules 的问题

我们平时的 `yarn` 命令执行后会生成一个 node_modules 文件夹。通过 node 的 resolve 算法，我们可以使用 node_modules 中按照的包。但是因为很多原因，这个过程非常的低效：

- node_modules 文件夹中包含海量的文件。`yarn` 命令中超过 70% 的时间都是花在这个文件夹上面。即使有了已经安装的文件也不行，包管理器要检查当前目录中的文件和最终文件的差异
- 由于生产 node_modules 文件夹是一个重 IO 的操作，所以包管理器没有太多的余地去优化他，因为他只是做了拷贝文件的操作
- node 并没有包的概念。他并不知道一个文件是否要被使用。完全可能出现这样的情况，在本地开发环境你的代码可以正常运行，但是由于在忘记向 package.json 文件中添加某个依赖，导致你的项目不能运行了
- 即使在运行时，node 也需要进行许多访问文件的操作去找出这个文件的实际位置在哪里（因为导入依赖时写的路径可以有多种解析的结果，node会一一尝试），这很浪费时间
- node_modules 的设计让包管理器很好的去减少重复包的引入（多个依赖的依赖是同一个包）。即使一些算法可以优化树状的文件结构（把所有的依赖都提升到最高级的目录），但是我们仍然可能会无法优化某些特定的情形，从而导致占用更多的存储空间，同一个包出现多次。

### [pnpm](https://pnpm.io/) 解决 node_modules 问题的尝试

这里先不说 yarn 是怎么解决这个问题的，我们先来看看社区的一些其他的解决方案，pnpm 就是这样的一个库。他也是一个包管理的库。

他和 yarn 以及 npm 不同，他的 node_modules 的文件夹结构仍然是树形的。这样做有一个好处，就是我们访问 node_modules 下的包时，我们就仅仅可以访问我们添加在 package.json 的依赖中的包，而不像 yarn 和 npm 一样，提升到最高层级的依赖的依赖也可以访问。

但是这样就有一个我们之前提到过的问题，当多个包依赖同一个包，或者同一个包的不同版本时，会大量增加磁盘的使用。为了解决这个问题，pnpm 在存储多个不同版本的同一个包时，仅仅会多存储那些发生改变过的文件。就比如对于一个包的 v1 和 v2，他们各自有 100 个文件，其中 99 个文件内容相同，1 个文件不同，pnpm 只会存储 101 个文件，99 个相同的文件，和两个版本各自 不同的文件。

除此之外所有项目的依赖的文件都将存储于同一个位置，他们通过 hard link 的方式存放到各自项目的 node_modules 文件夹下。

### PnP 的解决方式

yarn 知道依赖的包的安装位置，不仅如此，他还负责安装这些包。所以，为什么不是由 yarn 来负责定位寻找和管理依赖的安装位置呢？

从 yarn2.0 开始，yarn 会生成一个 .pnp.cjs 的文件，这个文件包含了以下两个映射关系：

- 某个特定版本包名字映射他的安装位置
- 某个特定版本包名字映射他所有的子依赖的特定版本的包名字

通过这些信息， yarn 可以告诉 node 应该在哪里找到某一个包，只要他们是依赖树的一部分。所有的包都被安装在 `.yarn/cahce` 文件夹中。每个包都是一个 .zip 的文件。所以文件的数量大大减少，可以提交到 git 仓库，实现 Zero-Installs。通过 zip loading 的特性，我们可以访问这些 zip 文件。

这个解决方式有以下的优点

- 安装几乎是瞬时的，yarn 只需要生成一个文件而不是原来的成千上万的文件。主要的瓶颈由原来的磁盘性能变成了依赖的数量
- 由于减少了 IO 操作的数量，安装过程变的更加可靠。尤其是在 Windows 上，批量读写文件可能因为类似 Windows Defender 的工具引起未知的错误。node_modules 的重 IO 操作更可能会导致失败
- 对于依赖树完美的优化
- 生成的 pnp.cjs 文件可以被提交到你的仓库作为 Zero-Installs 的一部分
- 应用启动速度更快。node resolution 不需要像之前一样遍历整个文件系统

### 使用方法

无论是在 yarn1 还是在 yarn2 中都可以使用 pnp 特性。

#### yarn1

首先，你需要 yarn 1.12+ 版本

然后只需执行以下命令即可开启 PnP 特性

```
yarn --pnp
```

#### yarn2

yarn2 和 yarn1 不同。对于每一个单独的项目，都需要指定并锁定使用的 yarn 的版本。

这也是一个可以理解的事情。我们的项目是会在不同的环境上运行的，这些环境可能各自安装了不同版本的 yarn。这些 yarn 的版本之间完全可能会存在一定的 break changes。所以说，为了保证项目的稳定性，不仅仅依赖的版本应该被锁定，yarn 这个依赖管理工具的版本也应该被锁定。

我们先运行以下命令设定 yarn 的版本

```
yarn set version berry
```

当然我们也可以把 berry 换成其他的值，来设定成其他的版本。

然后初始化 yarn 设置

```
yarn init
```

在 gitignore 文件中加入一下代码

```
.yarn/*
!.yarn/cache
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/sdks
!.yarn/versions
```

最后就可以通过 `yarn add` 安装依赖了。

### 生成的文件说明

- .yarn/cache 和 .pnp.\*：这两个可以被 gitignore，但是这就丧失了 Zero-Installs 的功能。.yarn/cahce 是包保存的位置，.pnp.\* 是运行时文件，保存了包存储位置的映射信息
- yarn/install-state.gz：这是一个不需要被 commit 的优化文件，他保存了你当前项目的状态，如果你运行下一个命令的时候，通过他可以节省一些时间
- .yarn/patches: 保存了通过 `yarn patch-commit` 命令生产的 patchfiles
- .yarn/plugins，.yarn/releases：分别保存了我们在这个项目中引入的 yarn 插件以及我们使用 `yarn set version` 命令指定的管理当前项目的 yarn 的版本的源文件。这保证了我们在不同环境运行时使用的 yarn 都是一致的
- .yarn/sdks：这里保存了由 `@yarnpkg/sdks` 生成的编辑器要用的 sdk，这个可以被 gitignore，不过这样你克隆项目的时候就需要重新安装这些 sdk
- .yarn/unplugged：这个文件夹保存了经过 unplug 之后的包
- .yarn/versions：被 [version plugin](https://yarnpkg.com/features/release-workflow) 使用存储一些包的发布信息
- yarn.lock：保证各个包的版本以及下载位置的稳定
- .yarnrc.yml：本项目 yarn 的配置文件

### 使用全局缓存

在 `.yarnrc.yml` 文件中加入以下配置

```
enableGlobalCache: true
```

这时，我们的依赖将不会被拷贝到本项目的 `.yarn/cache` 文件夹下，而是直接使用 yarn 的全局缓存文件夹

### pnp loose mode

我们知道，yarn 在 node_modules 目录中把属性结构的依赖树拉平成了单层的依赖列表，但是这个行为的结果是非标准化的且不稳定的，也就是说同一个依赖的结果运行多次是不稳定的。所以 PnP 禁止 require 那些不在 package.json 文件中的依赖。也就是说依赖的依赖是默认禁止被 require 的。但是，某些包可能会由此引发一些问题。

为了解决这个问题，yarn 推出了 loose mode。我们会生成一个 fallback pool，这个 pool 中包含了所有被提升到顶部的依赖。当我们 require 的东西不在依赖列表之中，我们会在 fallback pool 中寻找这个依赖。由于被提升到顶部的依赖的版本是不确定的，所以我们会报一个 warning。

要开启 loose mode，只需要在 `.yarnrc.yml` 文件中加入一下代码即可。

```
pnpMode: loose
```

除此之外，要确保 `.yarnrc.yml` 文件中 `nodeLinker: "pnp"` 设置没有被修改（这是默认值）

### 其他注意事项

由于 pnp 特性开启后，对于依赖的寻找是由 yarn 负责而不能直接由 node 负责，所以说在运行 node 脚本的时候不能直接通过 `node xxx.js` 来运行，否则可能会找不到依赖包的位置，需要引入pnp.cjs这个文件。可以由以下几个方法代替

- 执行 `yarn node xxx.js`
- 在要执行的脚本开头加入一行 `require("./.pnp.cjs").setup()` 的代码
- 执行 `node -r ./.pnp.cjs ./xxx.js`
- 执行 `NODE_OPTIONS="--require $(pwd)/.pnp.cjs" node ./xxx.js`

### 兼容性

目前大多数包都可以兼容 PnP 特性，但是仍然有一些例外。

其中 webpack4.x 可以通过 [pnp-webpack-plugin](https://github.com/arcanis/pnp-webpack-plugin) 兼容。

以下是一些常见的但是不兼容的包的列表

- angular
- flow
- react native
- pulumi
- VSCode Extension Manager
- Hugo
- ReScript

不过这并不意味着你在使用这些包的时候你就不能使用 PnP 了。你可以使用 [node-modules plugin](https://github.com/yarnpkg/berry/tree/master/packages/plugin-nm)，具体操作流程可以看这个[教程](https://yarnpkg.com/getting-started/migration#if-required-enable-the-node-modules-plugin)

### [缺陷](https://yarnpkg.com/features/pnp#caveat)

由于我们在发生 resolution error 的时候发出的是 warning 而不是 error，所以应用无法捕获到。这意味着如果在一个 try/catch 中 require 一个 peer dependency，那么将报一个 warning，即使他不该这样（这里我不是很理解，照理应该是throw error，但这里只报 warning，怎么会没有影响？贴一个原文链接，可以自己领会）。唯一的 runtime 影响就是这会让人们感到困惑，但是我们可以安全的忽略他。

### yarn unplug

这个命令用于调试第三方包，以 lodash 为例，我们运行 `yarn unplug lodash`，lodash 包将从 `.yarn/cache` 文件夹下消失，转移到 `.yarn/unplugged` 文件夹下，并且文件由原来的 .zip 格式变成一个正常的文件夹，这样我们可以直接修改这个文件夹中的内容对第三方包进行调试。

除此之外，将会在 `package.json` 文件中新增一段配置

```
"dependenciesMeta": {
  "lodash@4.17.21": {
    "unplugged": true
  }
}
```

如果调试完之后想要取消，在 `package.json` 文件中删除这段配置，然后执行 `yarn` 命令即可
