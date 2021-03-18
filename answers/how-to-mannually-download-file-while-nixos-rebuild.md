# 怎么解决 nixos-rebuild 过程中下载失败，能手动下载好文件然后继续么?

`nix-store --add-fixed <path>` 与 `nix-store --add <path>` 可以将目录加入 nix store。我们可以手动下载文件后使用这些方式来加载文件。

[参考](https://discourse.nixos.org/t/how-to-manual-add-tarball-for-fetchurl-build/6180/2)

## nix 的 daemon 模式使用代理

对于 systemd，可以通过在 nix-daemon 的 `overrides.conf` 或 service 内写入环境变量来使用代理。在 NixOS 中，可以使用 `networking.proxy` 相关参数进行设置。亦或使用 `systemd.services.nix-daemon.environment.<proto>_proxy` 等方式。

参考：[nixos manual/install behind proxy](https://nixos.org/manual/nixos/stable/#sec-installing-behind-proxy)

另注：在 nixpkgs 中，可以这样子使用代理是因为 fetcher 中加入了相关的 impureEnvVars
