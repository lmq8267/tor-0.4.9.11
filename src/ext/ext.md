@dir /ext
@brief 外部维护的代码

"ext" 目录包含在其他地方编写的代码，这些代码在我们希望构建的环境中无法可靠地作为库使用，因此我们将其随 Tor 一起发布。

通常情况下，您不应编辑这些代码：我们不是这些代码的维护者。相反，您应该向上游提交补丁。

OpenBSD_malloc_Linux.c:

> OpenBSD malloc 实现，移植到 Linux。仅在向 configure 脚本传递 --enable-openbsd-malloc 时使用。

strlcat.c
strlcpy.c

> strlcat 和 strlcpy 的实现，它们是 strcat 和 strcpy 的更安全替代品。这些是非标准函数，一些 libc 实现出于原则性原因拒绝添加它们。

ht.h

> Niels Provos 风格的哈希表实现（类似 tree.h）。与 Libevent 共享。

tinytest.c tinytest.h
tinytest_demos.c
tinytest_macros.h

> 一个单元测试框架。https://github.com/nmathewson/tinytest

tor_queue.h

> 来自 OpenBSD 的 sys/queue.h 的副本。我们保留自己的副本而不是使用 sys/queue.h，因为某些平台没有 sys/queue.h，而有此文件的平台之间也存在不兼容的差异。（有还是没有 CIRCLEQ？使用 SIMPLQ 还是 STAILQ？）我们还将标识符重命名为 TOR_ 前缀，以避免与系统头文件冲突。

curve25519_donna/*.c

> Adam Langley 的 curve25519-donna 基本可移植的 curve25519 实现的副本。

csiphash.c
siphash.h

> Marek Majkowski 的 siphash 2-4 实现，这是一种安全的带密钥哈希算法，用于避免针对哈希表的基于碰撞的 DoS 攻击。

trunnel/*.[ch]

> Trunnel 的头文件和运行时代码，Trunnel 是一个用于生成编解码二进制格式代码的系统。

ed25519/ref10/*

> Daniel Bernsten 的可移植 ref10 ed25519 实现。公共领域。

ed25519/donna/*

> Andrew Moon 的半可移植 ed25519-donna ed25519 实现。公共领域。

keccak-tiny/

> David Leon Gil 的可移植 Keccak 实现。CC0 许可证。

readpassphrase.[ch]

> 来自 OpenSSH portable 的可移植 readpassphrase 实现，版本 6.8p1。

timeouts/

> William Ahern 的分层定时器轮实现。MIT 许可证。

mulodi/

> 包含来自 LLVM compiler_rt 的溢出检查 64 位有符号整数乘法。出于某种原因，这在许多地方的 32 位 libclang 中缺失。双重许可，MIT 许可证和类 BSD 许可证；参见 mulodi/LICENSE.TXT。
