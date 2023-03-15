对于国内用户，可以考虑直接从镜像站安装 nix:  https://mirrors.tuna.tsinghua.edu.cn/help/nix/

安装好 nix 之后，可以参考以下配置，告知 nix 默认从镜像站下载:  https://mirrors.ustc.edu.cn/help/nix-channels.html

但是 nix 包在构建过程中，仍然可能访问除 nixpkgs（已经配置走镜像站）之外的地址，所以仍然可能需要尽快解决网络问题。
