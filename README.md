NixOS FAQ
=========

NixOS 常见问题解答, 如有新问题请加入 [Telegram Group](https://t.me/nixos_zhcn) 寻求帮助


#### Nix/NixOS/HomeManager 使用问题


1. [怎么升级 NixOS 版本?](answers/how-to-upgrade-nixos-version.md)
  
2. [如何回收 /nix/store 占用的磁盘空间?](answers/how-to-reclaim-disk-space-from-nix-store.md)

3. [怎么解决 nixos-rebuild 过程中下载失败，能手动下载好文件然后继续么?](answers/how-to-mannually-download-file-while-nixos-rebuild.md)

4. [哪里可以查看 NixOS module 选项的文档？](answers/where-to-find-doc-for-nixos-module-options.md)

5. [哪里可以查看 home manager 选项的文档？](answers/where-to-find-doc-for-home-manager-options.md)

6. [有办法在键入的命令未安装时自动安装并执行么？](answers/how-to-auto-run-command.md)

7. [Kernel 与 Glibc 的文档在哪里？](answers/where-to-find-doc-for-kernel.md)

#### Nix/NixOS 内部机制

1. [什么是 NixOS module ?](answers/what-is-nixos-module.md)

2. [什么是 nixpkgs Overlays？](answers/what-is-nixpkgs-overlays.md)

3. [什么是 NIX_PATH 和 \<path\> 语法?](answers/what-is-nix-path-and-angle-brackets-syntax.md)

4. [什么是 Nix Flakes?](answers/what-is-nix-flakes.md)

5. [为什么 nix-env 安装非默认的 outputs 后没有导出 PATH 等相关的环境变量？](anserwer/why-does-not-nix-env-export-path-for-non-default-outputs.md)


#### Nix 语言和设计模式相关

1. [哪里可以检索 Nix 的 builtins 函数文档？](answers/where-to-find-doc-for-nix-builtins.md)

2. [Nix 代码里经常看到的 callPackage 是怎么回事?](answers/what-is-call-package-in-nix.md)

3. [Nix 代码里经常看到的 xxx.override 是怎么回事？](answers/what-is-override-in-nix-code.md)


#### Nix 打包问题

1. [如何对 Nix 打包进行调试?](answers/how-to-debug-nix-packaging.md)

#### Nix Server 问题汇总

1. [hash mismatched](answers/nix-serve-hash-mismatch.md)
