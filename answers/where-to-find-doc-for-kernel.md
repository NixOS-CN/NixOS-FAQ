# Kernel 与 Glibc 的文档在哪里？

`man` 指令为单独的包，随系统安装，但内部不会包含 kernel 与 glibc 的文档。这些文档包含在 `nixpkgs.man-pages` 内。此外，glibc 的文档也可在 `nixpkgs.glibcInfo` 中找到。
