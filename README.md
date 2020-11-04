# NixOS FAQ
NixOS 常见问题解答  
如有新问题请加入 [Telegram Group](https://t.me/nixos_zhcn)

<details><summary> 怎么回收磁盘存储空间? </summary>
<p>

先删除 gc root （+5 表示保留最近的 5 个版本）

```sh
sudo nix-env --profile /nix/var/nix/profiles/system --delete-generations +5
```

然后再执行 gc 操作

```
nix-collect-garbage
```

如果想进一步节省磁盘空间，可以考虑用如下命令将相同的文件硬链接到同一份，但这个步骤可能会花费比较长的时间

```
nix-store --optimise
```

</p>
</details>


<details><summary>1. 怎么升级 NixOS 大版本?</summary>
<p>
  
### 关于 system.stateVersion

修改这个选项**不会**升级系统，如果你没弄清楚这个选项是[做什么的](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/misc/version.nix#L56), 请**不要**修改。

### 操作指南

以从 20.03 升级到 20.09 为例:

#### 1. 替换 nix-channel

在没有改动默认设置的情况下，root 默认拥有 `nixos` 这一 channel ，其 url 指向系统初次安装时使用的版本。

假设初次安装时使用20.03，则执行以下命令：

```sh
sudo nix-channel --list
```
应该会输出：

```
nixos https://nixos.org/channels/nixos-20.03
```

这时候执行：

```sh
sudo nix-channel --remove nixos
# --add 默认会覆盖已存在的channel name，上述 --remove 可以省略。
sudo nix-channel --add https://nixos.org/channels/nixos-20.09 nixos
```

完成后再检查一下输出：

```sh
sudo nix-channel --list
```

就应该看到 URL 已经被替换成了新版本的地址：

```
nixos https://nixos.org/channels/nixos-20.09
```

#### 2. 更新 channel

```sh
sudo nix-channel --update
```
这一步类似的作用是更新本机channel中的nix表达式，类似 `sudo apt-get update` , 参考 [Wiki](https://nixos.wiki/wiki/Cheatsheet)。

更多说明：[channel 所有者](#channel-所有者)

#### 3. 重新构建 (Rebuild) 系统

然后像往常一样, 重现构建系统：

```
sudo nixos-rebuild boot
```
使用 boot 可以避免部分 services 在 rebuild 后重启失败的问题，但需要重启系统。注： rebuild 不会使用 kexec 等技术自动切换内核。

如果仅更新 nixos ，可以跳过第2部的更新 channel ，执行 `sudo nixos-rebuild boot --upgrade` ，它会自动更新 nixos channel 并执行后续操作。更多说明： [nixos-rebuild --upgrade](#nixos-rebuild---upgrade)

更多说明：[rebuild 脚本](#rebuild-脚本)

#### 4. 重启系统

保存好进行中的工作, 然后重启：

```sh
reboot
```

然后 check 一下版本号是否最新：

```sh
nixos-version
```

现在应该能看到类似下面这样的输出:

```
20.09.1469.13d0c311e3a (Nightingale)
```

### 原理简介

#### channel 所有者

shell 命令使用 sudo 描述代替 root 用户，表示需要使用 root 所有的配置文件或 root 权限。`nixos-rebuild switch` 等需要 root 的写入权限。而 nixos-rebuild 读取的是 root 所拥有的 channel 。如果使用其它用户执行 `nixos-rebuild build`，则就是以这个用户的 channel 配置为准。但需要注意到，其它用户隐式包含了 root 的 channel 。

#### rebuild 脚本

`nixos-rebuild` 是一个 shell 脚本，在执行 `nixos-rebuild boot` 时，核心其实是执行了如下指令：
```sh
system=$(nix-build '<nixpkgs/nixos>' --no-out-link -A system)
$system/bin/switch-to-configuration boot
```
其中 `<nixpkgs/nixos>` 是 nix 中的特殊[语法](https://nixos.org/manual/nix/stable/#env-NIX_PATH)。简单来说，默认情况下 NixOS 中 root 的 `NIX_PATH` 环境变量的值为:
```
nixpkgs=/nix/var/nix/profiles/per-user/root/channels/nixos:nixos-config=/etc/nixos/configuration.nix:/nix/var/nix/profiles/per-user/root/channels
```
因此 sudo 执行 `nixos-rebuild boot` 时 `<nixpkgs/nixos>` 会被展开为 `/nix/var/nix/profiles/per-user/root/channels/nixos/nixos`, 而这个路径正是 root 的名为 `nixos` 的 channel 存放 nix 表达式的位置，因此替换更新 channel 之后再执行 `nixos-rebuild` 就会从新版本的 nix 表达式中构建系统。

同理用户可以修改 `NIX_PATH` 或者使用 `-I` 选项修改 `nixpkgs` 指向的路径，从而使用本地任意版本的 nixpkgs repo 构建系统。

##### nixos-rebuild --upgrade

`nixos-rebuild --upgrade` 实际上是先更新执行用户的 nixos channel。然后检查 root 用户的 channel ，如果一级目录中包含 `.update-on-nixos-rebuild` ，则会更新这个 channel 。
</p>
</details>

<details><summary>2. 如何对Nix打包进行调试?</summary>
<p>

### 避免 GC 构建的文件
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

</p>
</details>

<details><summary>3. Nix打包时常见错误</summary>
<p>

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

</p>
</details>
