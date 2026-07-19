# 实用工具

这些工具对于参与 Tor 开发并非严格必需，但它们可以帮助追踪 bug。

## Travis/Appveyor CI

这是 CI（持续集成）服务。

看起来像这样：
* https://travis-ci.org/torproject/tor
* https://ci.appveyor.com/project/torproject/tor

Travis 在 Linux 上构建并运行测试，最终也会支持 macOS（#24629）。
Appveyor 在 Windows 上构建并运行测试（使用 Windows Services for Linux）。

当向 torproject/tor 提交 Pull Request 时会自动运行。你也可以为你的 fork 设置它，以便在 PR 之外构建提交：

1. 注册 GitHub：https://github.com/join
2. fork https://github.com/torproject/tor：
   https://help.github.com/articles/fork-a-repo/
3. 按照 https://docs.travis-ci.com/user/getting-started/#To-get-started-with-Travis-CI 进行操作。
   跳过涉及 `.travis.yml` 的步骤（我们已经有一个了）。
4. 前往 https://ci.appveyor.com/login ，使用 github 登录，并选择 "NEW PROJECT"

构建结果应该会显示在 travis-ci.com 网站上以及 IRC 的 OFTC #tor-ci 频道。如果没有，请咨询 #tor-dev（同样在 OFTC 上）。

## Jenkins

这是 CI/构建服务。看起来像这样：https://jenkins.torproject.org

在提交合并到 git.torproject.org 时自动运行。我们对主分支和所有受支持的 tor 版本进行 CI 构建。我们还从主分支构建每夜 debian 软件包。

构建 Linux 和 Windows 交叉编译版本。运行 Linux 测试。

构建结果应该会显示在 jenkins.torproject.org 网站上以及 IRC 的 OFTC #tor-bots 频道。如果没有，请咨询 #tor-dev（同样在 OFTC 上）。

## Valgrind

```console
$ valgrind --leak-check=yes --error-limit=no --show-reachable=yes src/app/tor
```

（请注意，如果你看到大量的 openssl 警告，你还需要向 valgrind 传递 `--undef-value-errors=no` 参数，或者使用 `-DPURIFY` 重新编译你的 openssl。）

## Coverity

Nick 定期对 Tor 代码库运行 Coverity 静态分析器。

预处理器宏 `__COVERITY__` 用于处理 Coverity 检测到但我们允许存在的行为实例。

## clang 静态分析器

clang 静态分析器可以通过 Xcode（开发中）或命令行构建在 Tor 代码库上运行。

预处理器宏 `__clang_analyzer__` 用于处理 clang 检测到但我们允许存在的行为实例。

## clang 运行时消毒器

要使用 clang Address 和 Undefined Behavior 消毒器构建 Tor 代码库，请参阅文件 `contrib/clang/sanitize_blacklist.txt`。

clang 检测到但我们允许存在的行为的预处理器解决方案也记录在该黑名单文件中。

## 运行 lcov 进行单元测试覆盖率分析

lcov 是一个生成美观 HTML 测试覆盖率报告的工具。
要生成这样的报告：

```console
$ ./configure --enable-coverage
$ make
$ make coverage-html
$ $BROWSER ./coverage_html/index.html
```

这将运行 Tor 单元测试套件 `./src/test/test` 并在 `./coverage_html/` 目录下生成 HTML 覆盖率代码报告。要更改输出目录，请使用 `make coverage-html HTML_COVER_DIR=./funky_new_cov_dir`。

使用 lcov 进行覆盖率差异比较目前尚未实现，但正在研究中（截至 2014 年 7 月）。

## 运行单元测试

要快速运行 Tor 附带的所有测试：

```console
$ make check
```

只运行快速单元测试：

```console
$ make test
```

有选择地运行部分测试（以下方式可以任意组合）：

```console
$ ./src/test/test <name_of_test> [<name of test 2>] ...
$ ./src/test/test <prefix_of_name_of_test>.. [<prefix_of_name_of_test2>..] ...
$ ./src/test/test :<name_of_excluded_test> [:<name_of_excluded_test2]...
```

运行所有测试，包括基于 Stem 或 Chutney 的测试：

```console
$ make test-full
```

运行所有测试，包括需要互联网连接的基于 Stem 或 Chutney 的测试：

```console
$ make test-full-online
```

## 运行 gcov 进行单元测试覆盖率分析

```console
$ ./configure --enable-coverage
$ make
$ make check
$ # 或者--- make test-full ? make test-full-online?
$ mkdir coverage-output
$ ./scripts/test/coverage coverage-output
```

（在 OSX 上，你需要使用 `--enable-coverage CC=clang` 来开始。）

如果上述操作不起作用：

   * 尝试使用 `--disable-gcc-hardening` 配置 Tor
   * 你可能需要在运行 `./configure` 之后执行 `make clean`。

然后，查看 `coverage-output` 中的 .gcov 文件。行首的 '-' 表示编译器未为该行生成任何代码。'######' 表示该行从未被执行过。带有数字的行表示该行被调用了对应次数。

有关如何阅读 gcov 输出的更多详细信息，请参阅 GCC 手册中的 [调用 gcov](https://gcc.gnu.org/onlinedocs/gcc/Invoking-Gcov.html) 章节。

如果你对 Tor 做了更改并想要获取另一组覆盖率结果，可以运行 `make reset-gcov` 来清除中间 gcov 输出。

如果你有两个不同的 `coverage-output` 目录，并且想要查看它们之间有意义的差异，可以运行：

```console
$ ./scripts/test/cov-diff coverage-output1 coverage-output2 | less
```

在此差异中，任何至少被访问过一次的行，覆盖率都会显示为 "1"，并且行号会被删除。这让你可以检查（可能）你真正想知道的内容：哪些未测试的行被修改了？是否有新的未测试行？

如果你运行 ./scripts/test/cov-exclude，它会将排除的未到达行标记为 'x'，将排除的已到达行标记为 '!!!'。

## 运行集成测试

我们有一套使用 Chutney 运行集成测试的初始脚本。要试用它们，请将 CHUTNEY_PATH 设置为你的 chutney 源代码目录，然后运行 `make test-network`。

我们还有使用 Stem 运行集成测试的脚本。要试用它们，请将 `STEM_SOURCE_DIR` 设置为你的 Stem 源代码目录，然后运行 `test-stem`。

## 性能分析 Tor

关于 Tor 性能分析的持续记录可以在以下地址找到：
https://pad.riseup.net/p/profiling-tor

## 使用 oprofile 对 Tor 进行性能分析

oprofile 工具（仅在 Linux 上运行！）可以告诉你 Tor 在哪些函数上花费了 CPU 时间，从而帮助我们识别性能瓶颈。

以下是一些基本说明

 - 使用调试符号构建 tor（如果你在构建过程中没有修改 CFLAGS，你可能已经有了）。
 - 使用调试符号构建你关心的所有库（可能你只关心 libssl，也许还有 zlib 和 Libevent）。
 - 将此 tor 复制到一个新目录
 - 将它使用的所有库也复制到该目录（`ldd ./tor` 会告诉你）
 - 设置 LD_LIBRARY_PATH 以包含该目录。`ldd ./tor` 现在应该会显示它正在使用该目录中的库
 - 运行该 tor
 - 重置 oprofile 计数器/启动它
   * `opcontrol --reset; opcontrol --start`，如果 Nick 没记错的话。
 - 过一段时间后，让它转储该目录中 tor 和所有库的统计数据
   * `opcontrol --dump;`
   * `opreport -l that_dir/*`
 - 获益

## 使用 perf 对 Tor 进行性能分析

这适用于正在运行的 Tor，并且需要 root 权限。

1. 决定你想要分析多长时间。先从（比如说）30 秒开始。如果有效，再尝试更长的时间。

2. 找到正在运行的 tor 进程的 PID。

3. 运行 `perf record --call-graph dwarf -p <PID> sleep <SECONDS>`

   （你可能需要以 root 身份执行此操作。）

   如果你使用的是较旧的 CPU、无法访问硬件分析事件、在虚拟机中或类似情况，你可能需要在上面的 perf record 行中添加 `-e cpu-clock` 作为选项。

4. 现在你有了一个 perf.data 文件。使用 `perf report
   --no-children --sort symbol,dso` 或 `perf report --no-children --sort
   symbol,dso --stdio --header` 查看它。看起来怎么样？

5a. 一旦你有了一个足够大的 perf.data 文件，你可以压缩它、加密它，
    然后发送给你最喜欢的 Tor 开发者。

5b. 或者你可能不想发送一个很大的 perf.data 文件。谁知道里面有什么！？
    这有点吓人。要生成一个不那么吓人的文件，你可以使用 `perf
    report -g > <FILENAME>.out`。然后你可以压缩它并将其放在某个公开的地方。

## 使用 gperftools（又称 Google-performance-tools）对 Tor 进行性能分析

这应该几乎可以在任何类 Unix 系统上工作。但它似乎与 RunAsDaemon 不兼容。

事先，请安装 google-perftools。

1. 你需要重新构建 Tor，修改链接步骤以在库中添加 `-lprofiler`。你可以在调用 `./configure` 时添加 `LIBS=-lprofiler` 来实现。

现在你可以运行启用了性能分析的 Tor，并使用 pprof 工具查看性能！有关更多信息，请参阅 gperftools 手册，但基本步骤如下：

2. 运行 `env CPUPROFILE=/tmp/profile src/app/tor -f <path/torrc>`。配置文件在 Tor 执行完毕之前不会被写入。

3. 运行 `pprof src/app/tor /tmp/profile` 以启动 REPL。

## 生成和分析调用图

0. 在 Linux 或 Mac 上构建 Tor，理想情况下使用 -O0 或 -fno-inline。

1. 克隆 'https://git.torproject.org/user/nickm/calltool.git/' 。
   按照该仓库中的 README 进行操作。

请注意，目前调用图生成器无法检测通过函数指针的调用。

## 让 emacs 正确编辑 Tor 源代码

Nick 喜欢在他的 .emacs 文件中添加以下代码片段：


    (add-hook 'c-mode-hook
          (lambda ()
            (font-lock-mode 1)
            (set-variable 'show-trailing-whitespace t)

            (let ((fname (expand-file-name (buffer-file-name))))
              (cond
               ((string-match "^/home/nickm/src/libevent" fname)
                (set-variable 'indent-tabs-mode t)
                (set-variable 'c-basic-offset 4)
                (set-variable 'tab-width 4))
               ((string-match "^/home/nickm/src/tor" fname)
                (set-variable 'indent-tabs-mode nil)
                (set-variable 'c-basic-offset 2))
               ((string-match "^/home/nickm/src/openssl" fname)
                (set-variable 'indent-tabs-mode t)
                (set-variable 'c-basic-offset 8)
                (set-variable 'tab-width 8))
            ))))


你会注意到它默认显示所有尾随空白。`cond` 测试检测文件是否是我经常编辑的几个 C 自由软件项目之一，并设置相应的缩进级别和制表符偏好。

如果你想试用这个，你需要将文件名正则表达式模式更改为匹配你存放 Tor 文件的位置。

如果你只用 emacs 编辑 Tor 而不编辑其他内容，你总是可以直接说：


    (add-hook 'c-mode-hook
        (lambda ()
            (font-lock-mode 1)
            (set-variable 'show-trailing-whitespace t)
            (set-variable 'indent-tabs-mode nil)
            (set-variable 'c-basic-offset 2)))


可能有更好的方法来做这件事。不，我们可能不会在文件中添加 emacs 相关内容。

## 构建标签文件（代码索引）

tor 中的许多函数使用 `MOCK_IMPL` 包装器进行单元测试。你的标签构建程序必须被告知如何处理此语法。

如果你使用 emacs，你可以使用 `make tags` 生成一个 emacs 兼容的标签文件。这将运行你系统上的 `etags`。Tor 的构建系统假设你使用的是 emacs 特定版本的 `etags`（在 Debian 的 `xemacs21-bin` 包中捆绑）。这与其他版本的 `etags`（如 Exuberant Ctags 提供的版本）不兼容。

如果你使用 vim 或 emacs，你也可以使用 Universal Ctags 使用以下语法构建标签文件：

```console
$ ctags -R -D 'MOCK_IMPL(r,h,a)=r h a' .
```

如果你使用的是较旧版本的 Universal Ctags，你可以改用以下方式：

```console
ctags -R --mline-regex-c='/MOCK_IMPL\([^,]+,\W*([a-zA-Z0-9_]+)\W*,/\1/f/{mgroup=1}' .
```

默认会生成一个 vim 兼容的标签文件。如果你使用 emacs，请添加 `-e` 标志以生成 emacs 兼容的标签文件。

## Doxygen

我们使用 'doxygen' 工具从源代码生成文档。以下是如何使用它：

  1. 每个应该被记录的文件都以如下内容开头

```
 /**
  * \file filename.c
  * \brief 文件的简短描述。
  */
```

  （Doxygen 会将任何以 /** 开头的注释识别为特殊注释。）

  2. 在你想要记录的任何函数、结构体、#define 或变量之前，添加如下格式的注释：

```
/** 用祈使句描述函数的操作。
 *
 * 使用空行进行段落分隔
 *   - 并且
 *   - 使用连字符
 *   - 进行
 *   - 列表。
 *
 * 将 <b>参数名</b> 写为粗体。
 *
 * \code
 *     place_example_code();
 *     between_code_and_endcode_commands();
 * \endcode
 */
```

  3. 确保将字符 `<`、`>`、`\`、`%` 和 `#` 转义为 `\<`、`\>`、`\\`、`\%` 和 `\#`。

  4. 要记录结构体成员，你可以使用两种形式：

```c
struct foo {
  /** 你可以将注释放在元素之前； */
  int a;
  int b; /**< 或者使用小于号将注释放在元素之后。 */
};
```

  5. 要从 Tor 源代码生成文档，请输入：

```console
$ doxygen -g
```

  这会生成一个名为 `Doxyfile` 的文件。编辑该文件并运行 `doxygen` 以生成 API 文档。

  6. 有关更多信息，请参阅 Doxygen 手册；本摘要只是浅尝辄止。

## 样式和最佳实践检查

我们使用脚本来检查源代码格式和样式中的各种问题。"check-spaces" 测试在本地级别检测大量编码风格违规。"check-best-practices" 测试查找违反某些复杂性指南的情况。

你可以通过异常文件（scripts/maint/practracker/exceptions.txt）告诉工具关于复杂性指南的例外情况。但在这样做之前，请考虑你是否应该修复根本问题。也许那个文件确实_太_大了。也许那个函数确实做了太多事情。（另一方面，对于稳定发行系列，有时最好保持不重构。）
