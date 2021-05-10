如何回收 nix 占用的磁盘空间?
==================================


关于 /nix/store
---------------

首先先删除不再需要的 [gc root](https://nixos.org/manual/nix/stable/#ssec-gc-roots)。例如，如果只需要保留最近5次构建的 NixOS 版本：


```sh
sudo nix-env --profile /nix/var/nix/profiles/system --delete-generations +5
```

以及同理删除各用户的旧 profile（下列命令以哪个用户执行就清理谁的 profile）

```sh
nix-env --delete-generations +5
```

另外，在 NixOS 上如果这次启动系统之后执行过 `nixos-rebuild switch`，那么可以考虑删除指向当前启动的版本的系统的 gc root：

```sh
rm /run/booted-system
```

NixOS 会在此保存本次开机时使用的系统版本，因为其中包括当前启动的内核的 module 等内容，如果清理掉可能导致在下次重启前 `modprobe` 失败。

然后再执行 gc 操作

```sh
nix-collect-garbage
```

nix 会删除所有未被任何 gc root 依赖的路径。

以上两步操作可以使用 `nix-collect-garbage --delete-older-than 5d` 代替，但不同于上述二步，这里删除的是 `/nix/var/nix/profies` 下面所有的配置，而不仅有系统，并且只可使用时间。如果你使用 NixOS ，可以配置 `nix.gc` 相关选项达到自动清理的目的。如：

```nix
{ ... }:
{
  nix.gc = {
    automatic = true;
    options = "--delete-older-than 5d";
    dates = "Sun 19:00";
  };
}
```

如果想进一步节省磁盘空间，可以定期执行 `nix-store --optimise` ，这个命令通过硬链接相同内容的文件的方式减少磁盘占用。另外，可以在 `nix.conf` 中添加如下设置，这样 nix 在每次向 nix store 写入时都会执行该优化：
```
auto-optimise-store = true
```

NixOS 用户可以在 `configuration.nix` 中加入如下设置：
```nix
{ config, ... }: {
  nix.autoOptimiseStore = true;
}
```

关于 /boot
----------

如果你的 /boot 分区满了, 你需要先清理旧的 profiles，然后执行 nixos-rebuild 以更新 /boot 分区并释放空间.

更多见： https://gsc.io/70266391-48a6-49be-ab5d-acb5d7f17e76-nixos/doc/nixos-manual/html/sec-nix-gc.html
