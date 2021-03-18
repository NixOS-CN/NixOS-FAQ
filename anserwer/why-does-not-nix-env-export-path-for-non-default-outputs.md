# 为什么 nix-env 安装非默认的 outputs 后没有导出 PATH 等相关的环境变量？

以 glib.dev 为例，`nix-env -iA glib.dev` 后并没有 `${glib.dev}/bin` 加入 `PATH`。

查看 `~/.nix-profile/manifest.nix` 发现其结构中存储包含 `glib.dev`，但于 `outputs` 与 `outputPath` 中使用的是 `bin`。相关参数可以看到 `outputsToInstall` 等。

查看 glib 在 nixpkgs 中的定义，我们可以看到 `meta.outputsToInstall` 中设置为 `bin`。

相关设计：[nix/commit](https://github.com/NixOS/nix/commit/9504bcf03c10b0e146b2969e1a2bcb279ee6c3d8)

## 关联问题：[kernel 文档](where-to-find-doc-for-kernel.md)

NixOS 中 `documentation` 相关 options 即通过使用 `environment.extraOutputsToInstall` 来完成。而在 home-manager 中也可以通过手工添加 `home.extraOutputsToInstall` 来完成。

注：kernel 文档问题源于这些文档是一个单独的包，而不在对应的 packages 分隔 outputs。
