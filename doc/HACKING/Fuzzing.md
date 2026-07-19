# 对 Tor 进行模糊测试

## 简单版本（不进行模糊测试，仅运行测试）

检出 fuzzing-corpora，并将 TOR_FUZZ_CORPORA 设置为指向你检出的位置。

要以确定性方式运行模糊测试用例，请使用：

```console
$ make test-fuzz-corpora
```

这实际上不会对 Tor 进行模糊测试！它只是在我们现有的模糊测试用例集上运行所有模糊测试二进制文件。

## 不同类型的模糊测试

目前我们支持三种不同类型的模糊测试器。

第一种是 American Fuzzy Lop (AFL)，一种通过 fork 目标二进制文件并通过 stdin 传递大量不同输入来工作的模糊测试器。它是最难设置的，所以我将在下面更详细地描述它。

第二种是 libFuzzer，一种基于 llvm 的模糊测试器，你将其作为库链接进去，它会反复运行目标函数。要使用这个，你需要安装一个较新的 clang 和 libfuzzer。然后，你只需使用 --enable-expensive-hardening 和 --enable-libfuzzer 进行构建。这将在 src/test/fuzz/lf-fuzz-* 中生成一组二进制文件。这些程序接收一系列包含模糊测试示例的目录作为输入。有关 libfuzzer 的更多信息，请参阅 https://llvm.org/docs/LibFuzzer.html

第三种是 Google 的 OSS-Fuzz 基础设施，它期望获取所有内容。有关更多信息，请参阅 https://github.com/google/oss-fuzz 和 projects/tor 子目录。你需要稍微摆弄一下 Docker 来测试这个；它旨在运行在 Google 的基础设施上。

在所有情况下，你都需要一些初始示例在模糊测试器启动时提供给它。在 "fuzzing-corpora" git 仓库中有一套示例。尝试将 TOR_FUZZ_CORPORA 设置为指向该仓库的检出

## 编写 Tor 模糊测试器

一个 tor 模糊测试框架应该包含：
* 一个 fuzz_init() 函数来设置任何必要的全局状态。
* 一个 fuzz_main() 函数来接收输入并将其传递给解析器。
* 一个 fuzz_cleanup() 函数来清理全局状态。

大多数模糊测试框架会产生许多无效输入 - 一个 tor 模糊测试框架应该在不崩溃或不正常运行的情况下拒绝无效输入。

但是，如果 tor 断言失败、触发 bug 或访问不应访问的内存，模糊测试框架应该崩溃。这有助于模糊测试框架检测"有趣的"案例。

## 使用 AFL 进行引导式模糊测试

American Fuzzy Lop 的源代码没有 HTTPS、哈希或签名，因此无法验证其完整性。话虽如此，你确实不应该在你关心的机器上进行模糊测试。

构建方法：
  从 http://lcamtuf.coredump.cx/afl/ 获取 AFL 并解压
  ```console
  $ cd afl
  $ make
  $ cd ../tor
  $ PATH=$PATH:../afl/ CC="../afl/afl-gcc" ./configure --enable-expensive-hardening
  $ AFL_HARDEN=1 make clean fuzzers
  ```

查找 ASAN 内存限制：（仅限 64 位）

在 64 位平台上，afl 需要知道 ASAN 使用了多少内存，因为 ASAN 倾向于分配大量的虚拟内存，然后实际上并不使用它。

阅读 afl/docs/notes_for_asan.txt 获取更多详细信息。

  从 https://jwilk.net/software/recidivm 下载 recidivm
  下载签名
  验证签名
  ```console
  $ tar xvzf recidivm*.tar.gz
  $ cd recidivm*
  $ make
  $ /path/to/recidivm -v src/test/fuzz/fuzz-http
  ```
  使用最终的 "ok" 值作为调用 afl-fuzz 时 -m 的输入
  （通常情况下，recidivm 会自动输出一个值，但在某些情况下，当内存限制太小时模糊测试框架会挂起。）

如果你不关心内存限制，你也可以直接在下面使用 "none" 代替内存限制值。


运行方法：

```console
$ mkdir -p src/test/fuzz/fuzz_http_findings
$ ../afl/afl-fuzz -i ${TOR_FUZZ_CORPORA}/http -o src/test/fuzz/fuzz_http_findings -m <asan-memory-limit> -- src/test/fuzz/fuzz-http
```

AFL 有多核模式，请查看文档了解详情。
你可能会发现附带的 fuzz-multi.sh 脚本对此很有用。

macOS (OS X) 需要稍多的准备工作，包括：
* 使用 afl-clang（或 llvm 目录中的 afl-clang-fast）
* 禁用外部崩溃报告（AFL 会引导你完成此步骤）

## 分类问题

崩溃通常是有趣的，特别是在使用 AFL_HARDEN=1 和 --enable-expensive-hardening 时。有时崩溃是由于框架代码中的 bug 导致的。

挂起可能是有趣的，但也可能是虚假的机器减速。
在报告挂起之前，请检查它是否可重现。有时，处理有效输入可能需要一秒钟左右，特别是在启用了模糊测试器和消毒器的情况下。

要查看 fuzz-http 对测试用例做了什么，可以这样调用：

```console
$ src/test/fuzz/fuzz-http --debug < /path/to/test.case
```

（模糊测试期间会禁用日志记录以提高模糊测试速度。）

## 报告问题

请使用 Tor 安全问题政策中的流程报告发现的任何问题：

https://gitlab.torproject.org/tpo/core/team/-/wikis/NetworkTeam/SecurityPolicy
