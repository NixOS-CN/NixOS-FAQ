见官方手册： https://nixos.org/manual/nix/stable/#env-NIX_PATH

简单说，Nix 表达式在被求值时，其中以尖括号包裹的路径表达式 （如： `<path>` ），会参考 NIX_PATH 环境变量的值，进行展开。

在新的基于 flakes 的实现中，已经不推荐这种用法了，因为这种机制让一部分状态泄漏到了环境当中。
