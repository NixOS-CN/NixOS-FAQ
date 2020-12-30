如何回收 /nix/store 占用的磁盘空间?
==================================


首先先删除不再需要的 [gc root](https://nixos.org/manual/nix/stable/#ssec-gc-roots)。例如，如果只需要保留最近5次构建的 NixOS 版本：


```sh
sudo nix-env --profile /nix/var/nix/profiles/system --delete-generations +5
```

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

