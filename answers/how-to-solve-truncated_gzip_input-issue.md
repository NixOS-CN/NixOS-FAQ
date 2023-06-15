解决 nixos-rebuild 时报错 truncated gzip input 的问题
---

现象:

```
$ sudo nixos-rebuild switch
[sudo] password for lc:
building the system configuration...
error: cannot get archive member name: truncated gzip input
(use '--show-trace' to show detailed location information)
```

原因分析:

大概率是网络原因， 导致 nixpkgs 的 tar.gz 文件在下载到一半时被终止，而 nix daemon 在重试的时候发现已经有 cache 就继续执行下一步，导致报错，所以删掉 cache 后再重试即可解决。

解决:

```
$ su
$ cd /root/.cache/
$ rm -rf ./nix
```
