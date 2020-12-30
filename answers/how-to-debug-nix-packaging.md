如何对Nix打包进行调试?
======================

### 避免构建的 .drv 文件被 GC

在 `nix.conf` 中添加如下设置：

```
keep-outputs = true
keep-derivations = true
```

它们可以避免在 GC 时删除你的 drv 和编译时依赖，从而减少打包时不必要的重复构建。更多说明参考文档 `man nix.conf` 。

### 构建失败时保留临时文件

`nix-build` 在构建时默认执行在临时目录（详细查询 `$NIX_BUILD_TOP` 相关文档）。临时目录通常是在 `/tmp` 中（详细查询 `TMPDIR, TEMPDIR, TMP, TEMP` 等）。 `--keep-failed` 选项可以在构建失败的时候，保留这些文件，这时你可以使用 `nix-shell` 并进行构建目录来排查错误。

更进一步，nixpkgs 提供了 `breakpointHook` 可以让你在构建失败的时候产生提示信息来指导你后续调试，即使用 `cntr attach` 。使用这一钩子的方法为将其加入 `nativeBuildInputs` 中，如果构建过程出错，会产生类似如下的信息（查看原脚本 `<nixpkgs/pkgs/build-support/setup-hooks/breakpoint-hook.sh>` ，会发现这是一个打印提示并 sleep 非常久）：

```
build failed in  with exit code 255
To attach install cntr and run the following command as root:

   cntr attach -t command cntr-/nix/store/<hash>-shell
```

保留此进程前提下，使用 root 执行上述命令，会进入一个容器环境的根目录，之后可以进入构建目录完成查错与修正。如果不希望在用户环境安装 cntr ，可以使用 `nix run nixos.cntr -c ...` 来运行。


Nix打包时常见错误
-----------------

### no such file or directory

这种错误经常发生打包闭源二进制文件上，其中 elf 头的 interpreter 不正确上。可使用 `patchelf --set-interpreter` 进行修复，或者使用 nixpkgs 的 `autoPatchelfHook` 来自动完成。主要原理是 elf 的 interpreter 一般为 `/lib64/ld-linux-x86-64.so.2` ，但在 NixOS 中，这一目录是不存在的。我们需要修改为当前系统使用 glibc 、你所使用的其它 glibc 或相关库中的 ld。

### Segment fault

当确保编译过程没有问题的情况下，可以尝试关闭 strip ，避免移去必要的内容。例：

```
# https://nixos.org/manual/nixpkgs/stable/#ssec-fixup-phase
stdenv.mkDerivation {
    # Other arguments
    dontStrip = true;
}
```

### jar 文件无法找到库

表现形式为某一个类无法找到的错误。

如果确信类定义存在，当 jar 内包含共享库文件时，此问题可能相同于 elf 文件的 rpath 问题，处理方法有两种。一是解包 jar 文件，使用 `patchelf` 将 rpath 修正，再打包回去。二是直接打包 `LD_LIBRARY_PATH` 。

