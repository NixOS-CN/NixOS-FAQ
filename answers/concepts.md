关于 Nix/NixOS 的核心概念介绍
==========================


作为软件构建系统的 Nix 
-------------------

能够以可复现 ( [Reproducible / r13y ](https://r13y.com/iso-gnome/index.html) ) 的方式完成软件的构建.

如果我们把软件的构建过程, 看作是一系列互相有依赖的任务的执行过程, 那么这个过程可以用一个有向无环图 (DAG) 来表示.

在 Nix 里, 这个 DAG 是通过一系列后缀为 .drv 的文件来存储的, 每个 .drv 文件记录了这个 DAG 图中的一个顶点以及它的邻边 (它的依赖项) 的信息.

这些 .drv 文件, 以及执行这些 .drv 文件的过程中所产生的文件, 都被存储在一个特殊的目录下, 我们称之为 nix store, 它的路径通常是 `/nix/store`



相关概念:

- [Derivation](): 简称 drv, 它是 nix 里面的一个核心概念,  它类似于 build plan 中的 node, 一个 drv 的由依赖关系构成的传递闭包, 包含了一个 build plan 的所有信息.
- [nix store](): 储存所有 nix 软件包, 以及构建这些软件包的过程中用到的所有文件的一个特殊目录, 一般是 `/nix/store`
- [nar](): 为了方便 nix 传输构建结果 (通常是二进制缓存文件) 而创造的一种跟 tar 类似的归档文件格式


Nix 编程语言
-----------

Nix 语言是一门函数式编程语言, 它的 [核心数据结构类似JSON](), 同时它基于Lambda演算提供了定义计算过程的能力, 通过这两方面的能力, 你可以构造出足够复杂的数据结构, 它确实是一门图灵完备的编程语言.

在 Nix/NixOS 中, 我们主要用 Nix语言 提供声明式描述一个构建过程 (drv) 的能力, 以及声明式描述构建选项 ( pkgxxx.override { ... }  )  的能力.

这个语言有以下特点:

- 支持 类似JSON 的数据结构
- 基于 Lambda 演算 提供定义计算过程的能力, 支持 first class function
- 动态类型
- 采用惰性求值策略
- 具体语法有些模仿命令式语言的赋值语句 (如可以写 `{ a.b.c = 1; }`, 它等价于 `{ a = { b = { c = 1; }; }; }`, 并且多个这样的 "赋值语句" 还能合并 ) , 但内核实际上是声明式的

大概就是这样子, 熟悉 Haskell 或 Lisp 系列的玩家只需要去官网看5分钟关于具体语法的 [文档](https://nixos.org/manual/nix/unstable/expressions/language-constructs.html) 即可, 这里不赘述。


作为包管理器的 Nix
----------------

当作为包管理器使用时, Nix 主要指 nix-env 命令, 它允许用户安装包, 你可以像 `apt-get install` 一样使用 `nix profile install`, 

但它仍然有很多特别之处:

- 不仅支持传统的 [命令式包管理](https://nixos.org/manual/nix/unstable/command-ref/new-cli/nix3-profile-install.html) 
- 也支持先进的 [声明式包管理](https://nixos.org/manual/nixpkgs/stable/#sec-declarative-package-management)
- 支持[回滚操作](https://nixos.org/manual/nix/unstable/command-ref/new-cli/nix3-profile-rollback.html)
- 支持灵活的编译/构建选项配置 ( pkg1.override {xxx} ),  因为是用一门语言来指定选项, 因此比任何其他 pm 更灵活得多.
- 同时支持源码发布本地构建 和二进制发布两种形式, 并且支持两种形式同时混合使用 (优先下载二进制缓存, 若无缓存则本地构建)


作为 Linux 发行版的 NixOS
-----------------------


既然 Nix 已经把我们在 Linux 世界里常用的软件都进行了打包，形成了自己的软件发布体系，那么，搞出一个 NixOS 其实就是水到渠成的事情了。

这里说一下它的一些特性：

- 不兼容 [FHS]() 规范，而是把所有软件都安装到 /nix/store/ 目录下
- 依赖 systemd 管理系统服务
- 通过修改 [声明式 (declarative) 的配置文件](https://nixos.org/manual/nixos/stable/index.html#sec-configuration-file) 即可完成大部分系统配置及系统管理操作 (包括各种服务的声明式配置, 甚至内核的配置都可以直接声明)
- 支持一键回滚, 若因更新导致了任何问题可以直接reboot从而立即回到上一次的系统，俗称“滚不挂”

总得来说, 使用 NixOS 的主要好处是 declarative & reproducable.

reproducable 意味着软件不会因为依赖库的版本的不确定性而导致软件运行结果不一致,

而 declarative 则能让你声明式地管理你的系统, 你可以持续地演进它, 用 Git 对它进行版本管理, 让他逼近你想要的样子, 而不用在每次重装OS的时候把之前的所有琐碎操作全都人肉 replay 一遍之前的操作.

这些都能帮助你避免很多状态相关的奇奇怪怪的无法复现的问题，从而加深你对系统本身的运作逻辑的理解，而不用在迷雾中前行。





官方包仓库 Nixpkgs
-----------------


通过 nix 构建的软件仓库, 包含大量的用于打包现存软件的 nix 脚本. 它的代码仓库是 github 上的一个中心化的 单体仓库 (mono repo), 有全世界很多开发者在贡献代码 (可能是全世界范围内贡献者最多的仓库).


实现 Reproducible 的最后一块拼图: Nix Flakes
-----------------------------------------

提供一种版本锁定机制 (锁定 git commit id), 以及去中心化 (对比于 nixpkgs 的中心化模式)  的仓库管理 (就是说你用 flake 就可以依赖 github 上的任何一个 repo 了而不再是只依赖 nixpkgs )

具体 flakes 机制产生的动机是什么, 解决了什么问题, 可以看 [这篇文章](https://www.tweag.io/blog/2020-07-31-nixos-flakes/)


其它
---

- [Home Manager](https://github.com/nix-community/home-manager): 基于 Nix 的配置文件管理器
- [sops-nix](https://github.com/Mic92/sops-nix): 为了解决 nix store 导致的数据隐私问题 (nix store 中的内容全局可读) 而进入的加密工具
