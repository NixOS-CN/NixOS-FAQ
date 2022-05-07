# 怎么解决 nixos-rebuild 过程中下载失败?

我们在执行 `sudo nixos-rebuild switch` 之类的命令时, 可能会遇到网络问题, 此时有以下几种解决方案

## 使用 Binary Cache 镜像仓库

可以通过额外的 CLI 参数来指定 Binary Cache 的地址

```
sudo nixos-rebuild switch --option substituters "https://mirror.sjtu.edu.cn/nix-channels/store"
```

也可以把以下配置加到 `configuration.nix` 使得**后续**每次都优先走国内镜像, **若需要本次立即生效还是看上面**

```
  #注意, 旧版本的 nixos 里, 该选项叫 nix.binaryCaches
  nix.settings.substituters = lib.mkBefore [
    "https://mirror.sjtu.edu.cn/nix-channels/store"
    #"https://mirrors.tuna.tsinghua.edu.cn/nix-channels/store"
    #"https://mirrors.bfsu.edu.cn/nix-channels/store"
  ];
```

(选其中一个即可, 用多个镜像并不一定就能让速度变快...)


## 手动下载文件

遇到个别文件下载失败的, 可以通过浏览器等途径, 手动下载文件, 然后用 nix-store 命令将它们加入到 nix store.

```
nix-store --add-fixed sha256 <path>
```

之后可以重试 nixos-rebuild 命令, 此时由于 nix store 中存在该文件, 所以就不会再走网络下载了.

[参考](https://discourse.nixos.org/t/how-to-manual-add-tarball-for-fetchurl-build/6180/2)

## nix 的 daemon 模式使用代理

对于 systemd，可以通过在 nix-daemon 的 `overrides.conf` 或 service 内写入环境变量来使用代理。在 NixOS 中，可以使用 `networking.proxy` 相关参数进行设置。亦或使用 `systemd.services.nix-daemon.environment.<proto>_proxy` 等方式。

参考：[nixos manual/install behind proxy](https://nixos.org/manual/nixos/stable/#sec-installing-behind-proxy)

另注：在 nixpkgs 中，可以这样子使用代理是因为 fetcher 中加入了相关的 impureEnvVars

## 手动指定代理

上述设置的方式不够灵活, 有时候我们只是想临时走代理 (而不是总是走代理), 这时可以通过下述方式临时指定代理

### 使用 proxychains

先[参考文档](https://nixos.org/manual/nixos/stable/options.html#opt-programs.proxychains.proxies)配置好 proxychains, 之后可以通过以下命令测试配置是否成功:

```
proxychains4 curl -v https://cache.nixos.org/nix-cache-info
```

执行需要 sudo 的命令时需要注意, proxychains 在跨越 sudo 时似乎不会生效, 所以需要先 sudo 再 proxychains, 例如:

```
sudo proxychains4 curl -v https://cache.nixos.org/nix-cache-info   
```

以上测试脚本可以成功执行后, 就可以重新尝试  nixos-rebuild 了

```
sudo proxychains4 nixos-rebuild switch
```

### 使用 http_proxy 等环境变量

例如先引入以下环境变量 (将protocol,ip和port改为自己的代理服务器配置)

```
export http_proxy='http://10.112.1.1:8118'
export https_proxy='http://10.112.1.1:8118'
```

然后测试

```
curl -v https://cache.nixos.org/nix-cache-info
```

执行需要 sudo 的命令时需要注意, sudo 默认不会带环境变量, 除非指定 -E 参数

```
sudo -E curl -v https://cache.nixos.org/nix-cache-info
```

以上测试脚本可以成功执行后, 就可以重新尝试  nixos-rebuild 了

```
sudo -E nixos-rebuild switch
```
