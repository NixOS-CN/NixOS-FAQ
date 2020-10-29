# NixOS FAQ
NixOS 常见问题解答  (若有新问题请咨询TG群: https://t.me/nixos_zhcn)


## 1. 怎么升级 NixOS 大版本?

### 关于system.stateVersion

修改这个选项**不会**升级系统，如果你没弄清楚这个选项是[做什么的](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/misc/version.nix#L56), 请**不要**修改。

### 操作指南

以从 20.03 升级到 20.09 为例:

#### 1. 替换 nix-channel

在没有改动默认设置的情况下，root默认拥有`nixos`这一channel，其url指向系统初次安装时使用的版本。

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
sudo nix-channel --add https://nixos.org/channels/nixos-20.03 nixos
```

完成后再check一下：

```sh
sudo nix-channel --list
```

就应该看到 URL 已经被替换成了新版本的地址：

```
nixos https://nixos.org/channels/nixos-20.09
```

#### 2. 更新channel

```sh
sudo nix-channel --update
```
这一步类似的作用是更新本机channel中的nix表达式，类似`sudo apt-get update`, 参考[Wiki](https://nixos.wiki/wiki/Cheatsheet)。

#### 3. Rebuild系统

然后像往常一样, 重新 build 系统：

```
sudo nixos-rebuild boot

```

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

`nixos-rebuild`是一个shell脚本，在执行`nixos-rebuild boot`时，核心其实是执行了如下指令：
```sh
system=$(nix-build '<nixpkgs/nixos>' --no-out-link -A system)
$system/bin/switch-to-configuration boot
```
其中`<nixpkgs/nixos>`是nix中的特殊[语法](https://nixos.org/manual/nix/stable/#env-NIX_PATH)。简单来说，默认情况下NixOS中root的`NIX_PATH`环境变量的值为:
```
nixpkgs=/nix/var/nix/profiles/per-user/root/channels/nixos:nixos-config=/etc/nixos/configuration.nix:/nix/var/nix/profiles/per-user/root/channels
```
因此sudo执行`nixos-rebuild boot`时`<nixpkgs/nixos>`会被展开为`/nix/var/nix/profiles/per-user/root/channels/nixos/nixos`, 而这个路径正是root的名为`nixos`的channel存放nix表达式的位置，因此替换更新channel之后再执行`nixos-rebuild`就会从新版本的nix表达式中构建系统。

同理用户可以修改`NIX_PATH`或者使用`-I`选项修改`nixpkgs`指向的路径，从而使用本地任意版本的nixpkgs repo构建系统。
