怎么升级 NixOS 版本?
====================

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

(注意: nixos 这个 channel name 有特殊性，不能替换成其他名字)


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
这一步类似的作用是更新本机channel中的nix表达式，类似 `sudo apt-get update` , 参考 [Wiki](https://wiki.nixos.org/wiki/Cheatsheet)。

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


##### 关于 system.stateVersion

修改这个选项**不会**升级系统，如果你没弄清楚这个选项是[做什么的](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/misc/version.nix#L56), 请**不要**修改。

