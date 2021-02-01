见这个帖子：https://www.reddit.com/r/NixOS/comments/6a1z7f/nixos_can_automatically_run_noninstalled_programs/

简单说：

在 shell 的 rc 文件里设置环境变量：

- 设置 NIX_AUTO_RUN=1 可以使得未找到的命令自动被运行（不会创建 gc root），
- 设置 NIX_AUTO_INSTALL=1 可以使得未找到的命令被安装并运行（会创建 gc root）。

如果找到的命令不唯一，则不会发生任何自动动作。
