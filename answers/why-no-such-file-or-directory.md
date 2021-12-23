如何在 NixOS 中运行为其他 Linux 发行版所编译的可执行文件
========================================================

在 NixOS 中执行 Linux 的可执行文件时, 为什么会出现明明文件存在, 却提示 "no such file or directory" 呢?

这通常是因为你运行了一个并非为 NixOS 编译的 Linux ELF 文件, 它们可能常常会依赖 FHS 约定而去查找 /lib 或 /usr/lib 下的动态链接库文件, 而这些路径在 NixOS 中则并不存在.

(所以实际上, 这时候提示的 "找不到文件" 中的 "文件" 是指的当前可执行文件所依赖的动态链接库文件, 而不是当前文件本身)

你可以通过 ldd 命令来确认该情况 (假设此时你要执行的是 /hello 这个可执行文件):

```
ldd /hello
```

如果存在无法从当前环境找到的动态链接库文件, 则会出现一些类似下面的输出行:

```
libssl.so.1.1 => not found
```

此时可以手动指定 LD_LIBRARY_PATH 环境变量以运行命令, 如:

```
LD_LIBRARY_PATH=/path/to/lib /hello
```

在 NixOS 中, 我们可以将 /path/to/lib 指定为 nix store 中的某个路径. 为了得到这个路径, 我们需要先确定相关 so 文件在 nixpkgs 中的包名.


要想根据 so 的文件名, 确认包含它的包名, 可以使用以下命令:

```
nix-locate --top-level libssl.so.1.1
```

注意这并不是一个一一对应关系, 而是一个多对多关系, 所以你需要根据经验在输出结果里找到最像的那一个, 比如以上命令可能输出:

```
$ nix-locate --top-level libssl.so.1.1
remarkable-toolchain.out                        432,416 x /nix/store/x5ir2k5n8ayii4kfghch4i25ynfrp8b8-remarkable-toolchain-3.1.2/sysroots/cortexa9hf-neon-remarkable-linux-gnueabi/usr/lib/libssl.so.1.1
remarkable-toolchain.out                        585,064 x /nix/store/x5ir2k5n8ayii4kfghch4i25ynfrp8b8-remarkable-toolchain-3.1.2/sysroots/x86_64-codexsdk-linux/usr/lib/libssl.so.1.1
remarkable2-toolchain.out                       432,416 x /nix/store/3zzp37ay0nwx7a3xn72x7x3jrfybs9zi-remarkable2-toolchain-3.1.2/sysroots/cortexa7hf-neon-remarkable-linux-gnueabi/usr/lib/libssl.so.1.1
remarkable2-toolchain.out                       585,064 x /nix/store/3zzp37ay0nwx7a3xn72x7x3jrfybs9zi-remarkable2-toolchain-3.1.2/sysroots/x86_64-codexsdk-linux/usr/lib/libssl.so.1.1
openssl.debug                                         0 s /nix/store/412pnlx9qgzcny3i3l1r8h6nb5k1z8z8-openssl-1.1.1l-debug/lib/debug/libssl.so.1.1
openssl.out                                     701,016 x /nix/store/sr16hq3pdq96w5kbbksnngad2czxfvys-openssl-1.1.1l/lib/libssl.so.1.1
koreader.out                                    585,016 r /nix/store/j1zj2p74j2wrh3q6hdplxxgyq8zx48wy-koreader-2021.09/lib/koreader/libs/libssl.so.1.1
clpm.out                                        701,016 x /nix/store/hff3vli7mp9kizl3g4pqjf1dil3fpdws-clpm-0.3.6/lib/clpm/libssl.so.1.1
```

而此时你要找的包名则是 `openssl`, 所以 good luck ...


之后你就可以设置以下环境变量使得 nix store 中的动态链接库被找到 (若提供多个包名需以空格分割):

```
LD_LIBRARY_PATH=$(nix eval --impure --raw --expr "with import <nixpkgs> {}; lib.makeLibraryPath [openssl]")
```

带着以上环境变量运行 ldd 命令则可以帮助进一步发现还有哪些 so 文件没有被找到:

```
LD_LIBRARY_PATH=$(nix eval --impure --raw --expr "with import <nixpkgs> {}; lib.makeLibraryPath [openssl]") ldd /hello
```

确认所有动态链接库都被找到后, 就可以运行原命令了:

```
LD_LIBRARY_PATH=$(nix eval --impure --raw --expr "with import <nixpkgs> {}; lib.makeLibraryPath [openssl]") /hello
```

