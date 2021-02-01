Hash mismatch in Downloaded Path
==========

问题说明
------
使用缓存服务时，可能出现 `hash mismatch importing path` 等类似的错误。

原因分析
------
其原因出在服务器过于频繁的 gc。导致数据库和 nar 信息不匹配。

注：nixpkgs 官方使用的服务器不会进行 gc，基本不存在这个问题。

解决方案
------
首先，最简单的方式就是不要 gc。或者避免频繁 gc[1]。

其次，我们可以通过删除或修改本地的 `$HOME/.cache/nix/binary-cache-v*.sqlite` 来修正 narinfo[2]。亦或是，使用 `nix.conf` 中的 `narinfo-cache-negative-ttl` 和 `narinfo-cache-positive-ttl` 来减少缓存过期时间[1,2]。

最后，这个问题在 2020 年 11 月提了一个补丁[3]。但还没有进入 stable 分支。

外部参考
------
1. [NixOS/nix#1855](https://github.com/NixOS/nix/issues/1885)
2. [nix.dev faq](https://nix.dev/faq.html#how-do-i-force-nix-to-re-check-whether-something-exists-at-a-binary-cache)
3. [nix-server commit](https://github.com/edolstra/nix-serve/commit/4bbd001d67ded3151cfa3c5391f1b7d93dbb2f67)
