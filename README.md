# NixOS-FAQ
NixOS 常见问题解答  (若有新问题请咨询TG群: https://t.me/nixos_zhcn)


### 怎么升级 NixOS 大版本?

以从 20.03 升级到 20.09 为例:

1. 替换 nix channel

首先查看

```
sudo nix-channel --list
```

比如看到

```
nixos https://nixos.org/channels/nixos-20.03
```

这时候执行

```
sudo nix-channel --remove nixos
sudo nix-channel --add https://nixos.org/channels/nixos-20.03 nixos
```

完成后再check一下

```
sudo nix-channel --list
```

就应该看到 URL 已经被替换成了新版本的地址

```
nixos https://nixos.org/channels/nixos-20.09
```

2. 重新 build 系统

先更新一下包

```
sudo nix-channel --update
```

然后像往常一样, 重新 build 系统, 不过这次带上 --upgrade 参数

```
sudo nixos-rebuild --upgrade boot
```

3. 重启系统
