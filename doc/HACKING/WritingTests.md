# 为 Tor 编写测试：一份不完整的指南

Tor 使用各种测试框架和方法论来尽量避免引入 bug。主要的测试方法包括：

   1. 用 C 语言编写的单元测试，随 Tor 发行版一起发布。

   2. 用 Python 2 (>= 2.7) 或 Python 3 (>= 3.1) 编写的集成测试，随 Tor 发行版一起发布。

   3. 用 Python 编写的集成测试，随 Stem 库一起发布。其中一些测试使用 Tor 控制器协议。

   4. 用 Python 和 SH 编写的系统测试，随 Chutney 包一起发布。这些测试通过在本地运行多个 Tor 实例并通过它们发送流量来工作。

   5. Shadow 网络模拟器。

## 如何运行这些测试

### 简单版本

要运行 Tor 自带的所有测试，请运行 `make check`。

要同时运行 Stem 测试，请从 git 仓库获取 stem，将 `STEM_SOURCE_DIR` 设置为检出目录，然后运行 `make test-stem`。

要同时运行 Chutney 测试，请从 git 仓库获取 chutney，将 `CHUTNEY_PATH` 设置为检出目录，然后运行 `make test-network`。

要运行以上所有测试，请运行 `make test-full`。

要运行以上所有测试，加上需要连接互联网才能运行的测试，请运行 `make test-full-online`。

### 运行特定的子测试

Tor 的单元测试被划分为独立的程序和几个打包的单元测试程序。

独立程序很容易运行。例如，要单独运行 memwipe 测试，只需运行 `./src/test/test-memwipe`。

要在单元测试程序内运行测试，可以指定测试的名称。字符串 ".." 可用作测试名称末尾的通配符。例如，要运行所有 cell format 测试，请输入 `./src/test/test cellfmt/..`。

许多需要修改全局状态的测试在分叉的子进程中运行，以避免相互干扰。但在调试一个失败的测试时，你可能希望在不分叉子进程的情况下运行它。为此，可以对单个测试使用 `--no-fork` 选项。（如果同时指定了多个测试，它们可能会相互干扰。）

你可以通过传递 `--debug`、`--info`、`--notice` 或 `--warn` 来开启单元测试中的日志记录。默认情况下只显示错误。

单元测试分为 `./src/test/test` 和 `./src/test/test-slow`。前者是那些应该在几秒内完成的测试；后者往往需要更多时间，可能包含 CPU 密集型操作、人为延迟等。

## 查找测试覆盖率

测试覆盖率是衡量你的测试实际访问了哪些行的指标。

当你使用 `--enable-coverage` 选项配置 Tor 时，它应该会构建带有单元测试覆盖率支持和一个特殊的 `tor-cov` 二进制文件的版本。

然后，运行你想查看覆盖率的测试。如果你有旧的覆盖率输出，可能需要先运行 `reset-gcov`。

现在你会在构建目录中找到一堆名为 `*.gcda` 的文件。为了从中提取覆盖率输出，请为它们创建一个临时目录并运行 `./scripts/test/coverage ${TMPDIR}`，其中 `${TMPDIR}` 是你创建的临时目录。这将为测试下的每个源文件创建一个 `.gcov` 文件，包含该文件的源代码，并标注测试中每行被命中次数的注释。（你需要安装 gcov。）

你可以通过运行 `./scripts/test/cov-display ${TMPDIR}/*` 来获取每个文件的测试覆盖率摘要。每行列出文件名、未覆盖的行数、未覆盖的行数和覆盖率百分比。

要获取每个_函数_的测试覆盖率摘要，请运行 `./scripts/test/cov-display -f ${TMPDIR}/*`。

有关使用 gcov 的更多详细信息，包括 scripts/test 中的辅助脚本，请参阅 HelpfulTools.md。

### 比较测试覆盖率

有时比较你正在编写的分支的测试覆盖率与另一个分支（例如 git master）的覆盖率是很有用的。但你不能直接对两个覆盖率输出运行 `diff`，因为每行实际执行的次数并不那么重要，而且也不是完全确定性的。

相反，请按照上面的说明对每个分支执行操作，为每个分支创建一个单独的临时目录。然后运行 `./scripts/test/cov-diff ${D1} ${D2}`，其中 D1 和 D2 是你想比较的目录。这将生成两个目录的差异，所有行被规范化为已覆盖或未覆盖。

要计算 D2 中新增或修改的未覆盖行数，你可以运行：

```console
$ ./scripts/test/cov-diff ${D1} ${D2}" | grep '^+ *\#' | wc -l
```

## 将行标记为不可达的

你可以使用特殊字符串 LCOV_EXCL_LINE 将特定行标记为不可达的。你可以用 LCOV_EXCL_START... LCOV_EXCL_STOP 将一段行标记为不可达的。请注意，旧版本的 lcov 不理解这些行。

你可以对 .gcov 文件进行后处理，通过运行 ./scripts/test/cov-exclude 将这些行设为"未到达"。它将排除的未到达行标记为 'x'，将排除的已到达行标记为 '!!!'。

注意：你不应该这样做，除非该行确实应该被实际代码 100% 不可达。

## 我应该编写什么类型的测试？

集成测试和单元测试是互补的：如果可以的话，确保你的代码同时被两者覆盖可能是一个好主意。

如果你的代码是非常底层的，并且其行为很容易用输入和输出之间的关系或一组状态转换来描述，那么它自然适合单元测试。（如果不是，请考虑重构它，直到大部分_适合_单元测试！）

如果你的代码为 Tor 添加了新的外部可见功能，最好有一个针对该功能的测试。这通常是集成测试更常发挥作用的地方。

## 单元测试和回归测试：这个函数是否按照预期工作？

Tor 的大多数单元测试是使用 "tinytest" 测试框架制作的。你可以在 tinytest 手册中查看使用指南：

    https://github.com/nmathewson/tinytest/blob/master/tinytest-manual.md

要添加这种新测试，可以编辑 `src/test/` 中现有的 C 文件，或在那里创建一个新的 C 文件。每个测试都是一个单独的函数，必须在文件末尾的表格中进行索引。我们使用标签 "done:" 作为所有测试函数的清理点。

如果你创建了一个新的测试文件，你需要：
1. 将新的测试文件添加到 include.am
2. 在 `test.h` 中，包含新的测试用例 (testcase_t)
3. 在 `test.c` 中，将新的测试用例添加到 testgroup_t testgroups

（确保在继续之前阅读 `tinytest-manual.md`。）

我在这里使用"单元测试"和"回归测试"这两个术语非常随意。

## 一个简单的例子

这是一个针对 util.c 中简单函数的测试函数示例：

```c
static void
test_util_writepid(void *arg)
{
    (void) arg;

    char *contents = NULL;
    const char *fname = get_fname("tmp_pid");
    unsigned long pid;
    char c;

    write_pidfile(fname);

    contents = read_file_to_str(fname, 0, NULL);
    tt_assert(contents);

    int n = sscanf(contents, "%lu\n%c", &pid, &c);
    tt_int_op(n, OP_EQ, 1);
    tt_int_op(pid, OP_EQ, getpid());

done:
    tor_free(contents);
}
```

如果你读过 tinytest 手册，这看起来应该很熟悉。这里需要注意的一点是我们使用了测试专用函数 `get_fname` 来生成一个相对于测试使用的临时目录的文件。你不需要删除该文件；测试完成后它会被移除。

还要注意我们在 `tt_int_op()` 调用中使用 `OP_EQ` 而不是 `==`。我们定义 `OP_*` 宏来代替二元比较运算符，以便分析工具可以更容易地解析我们的代码。（Coccinelle 非常不喜欢看到 `==` 用作宏参数。）

最后，请记住，按照惯例，Tor 定义的所有 `*_free()` 函数都定义为可以安全接受 NULL。因此，你不需要在清理块中写 `if (contents)`。

## 为测试暴露静态函数

有时你需要测试一个函数，但不想将其暴露在其通常模块之外。

为此，Tor 的构建系统编译每个模块的测试版本，暴露额外的标识符。如果你想将一个函数声明为 static 但可供测试使用，请使用宏 `STATIC` 代替 `static`。然后，确保在模块的头文件中有该函数的宏保护声明。

例如，`crypto_curve25519.h` 包含：

```c
#ifdef CRYPTO_CURVE25519_PRIVATE
STATIC int curve25519_impl(uint8_t *output, const uint8_t *secret,
        const uint8_t *basepoint);
#endif
```

`crypto_curve25519.c` 文件和 `test_crypto.c` 文件都定义了 `CRYPTO_CURVE25519_PRIVATE`，因此它们可以看到此声明。

## 停下来！这个测试真的在测试吗？

编写测试时，仅仅在你测试的代码的所有行上产生覆盖率是不够的：确保测试_真正测试_了代码非常重要。

例如，这里是一个针对 unlink() 函数（应该用于删除文件）的_糟糕_测试。

```c
static void
test_unlink_badly(void *arg)
{
    (void) arg;
    int r;

    const char *fname = get_fname("tmpfile");

    /* If the file isn't there, unlink returns -1 and sets ENOENT */
    r = unlink(fname);
    tt_int_op(n, OP_EQ, -1);
    tt_int_op(errno, OP_EQ, ENOENT);

    /* If the file DOES exist, unlink returns 0. */
    write_str_to_file(fname, "hello world", 0);
    r = unlink(fnme);
    tt_int_op(r, OP_EQ, 0);

done:
    tor_free(contents);
}
```

这个测试可能在 unlink() 上获得非常高的覆盖率。那么为什么它是一个糟糕的测试呢？因为它没有检查 unlink() *是否确实删除了指定的文件*！

记住，测试的目的是在代码按预期工作时成功，在否则时失败。尽量设计你的测试，使其尽可能检查代码的预期和文档化的功能。

## 用于隔离测试的模拟函数

通常我们想测试一个函数是否正常工作，但要测试的函数依赖于其他行为难以观察的函数，或者需要一个正常运行的 Tor 网络，等等。

要为此情况编写测试，你可以在单元测试运行时用测试桩替换底层函数。你需要将底层函数声明为 'mockable'，如下所示：

```c
MOCK_DECL(returntype, functionname, (argument list));
```

然后稍后将其实现为：

```c
MOCK_IMPL(returntype, functionname, (argument list))
{
    /* implementation here */
}
```

例如，如果你有一个'连接到远程服务器'函数，你可以将其声明为：

```c
MOCK_DECL(int, connect_to_remote, (const char *name, status_t *status));
```

当你以这种方式声明一个函数时，它在常规构建中会被正常声明，但当模块用于测试构建时，它会被声明为一个初始化为实际实现的函数指针。

在你的测试中，如果你想用临时替代品覆盖该函数，可以这样写：

```c
MOCK(functionname, replacement_function_name);
```

然后，你可以用以下方式恢复原始函数：

```c
UNMOCK(functionname);
```

有关更多信息，请参阅 `testsupport.h` 中此模拟逻辑的定义。

## 好的，但我的测试实际上应该做什么？

我们在上面讨论了"测试覆盖率"——确保你的测试访问每一行代码或每一个代码分支。但仅仅访问代码是不够的：我们想要验证它是正确的。

因此，在编写测试时，尽量使测试在任何正确的代码实现下都通过，而在代码不按预期工作时应该失败。

你可以编写"黑盒"测试或"白盒"测试。黑盒测试是你在不查看函数结构的情况下编写的测试。白盒测试是你在查看函数实现方式的同时实现的测试。

无论哪种情况，请确保考虑常见情况*和*边界情况；成功情况和失败情况。

例如，考虑测试这个函数：

```c
/** Remove all elements E from sl such that E==element.  Preserve
 * the order of any elements before E, but elements after E can be
 * rearranged.
 */
void smartlist_remove(smartlist_t *sl, const void *element);
```

为了好好测试它，你应该至少为以下所有情况编写测试。（这些将是黑盒测试，因为我们只查看函数的声明行为：

   * 从 smartlist 中移除一个存在的元素。
   * 从 smartlist 中移除一个不存在的元素。
   * 从 smartlist 中移除一个出现多次的元素。

你的测试应该验证其行为是否正确。至少，你应该测试：

   * 在调用函数之后，E 之前的其他元素是否保持相同的顺序。
   * 目标元素是否确实被移除了。
   * _只有_目标元素被移除了。

当你考虑边界情况时，你可以尝试：

   * 从空列表中移除一个元素。
   * 从包含该元素的单元素列表中移除一个元素。
   * 从包含该元素多个实例且没有其他内容的列表中移除一个元素。

现在让我们看一下实现：

```c
void
smartlist_remove(smartlist_t *sl, const void *element)
{
    int i;
    if (element == NULL)
        return;
    for (i=0; i < sl->num_used; i++)
        if (sl->list[i] == element) {
            sl->list[i] = sl->list[--sl->num_used]; /* swap with the end */
            i--; /* so we process the new i'th element */
            sl->list[sl->num_used] = NULL;
        }
}
```

根据实现，我们现在看到了三个更多需要测试的边界情况：

   * 从列表中移除 NULL。
   * 从列表末尾移除一个元素。
   * 从列表末尾以外的位置移除一个元素。

## 我的测试不应该做什么？

测试不应该需要网络连接。

尽可能地，测试不应该超过一秒钟。如果确实需要更长时间运行，请将测试放入 test/slow 中。

除非使用 `TT_FORK` 运行，否则测试不应修改全局状态：测试不应要求其他测试在它们之前或之后运行。

测试不应该泄漏内存或其他资源。要检查你的测试是否泄漏内存，请在 valgrind 下运行它们（有关如何操作的更多信息，请参见 HelpfulTools.txt）。

在可能的情况下，测试不应该过度适应实现。也就是说，测试应该验证文档化的行为已实现，但如果后来实现了其他允许的行为，测试不应崩溃。

## 高级技术：命名空间

有时，当你同时进行大量模拟操作时，将标识符隔离在单个命名空间中是很方便的。如果是 C++，我们已经有了命名空间，但对于 C，我们用宏和标记粘合来尽力实现。

我们在 `src/test/test.h` 中为此目的定义了一些宏。要使用它们，你将 `NS_MODULE` 定义为你标识符的前缀，然后使用其他宏代替标识符名称。有关更多文档，请参阅 `src/test/test.h`。

## 集成测试：从外部调用 Tor

一些测试需要从外部调用 Tor，不应该在与 Tor 测试程序相同的进程中运行。这样做的原因可能包括：

   * 测试从命令行运行时 Tor 的实际行为
   * 测试崩溃处理器是否正确记录了堆栈跟踪
   * 验证违反沙箱或能力要求是否确实会导致程序崩溃
   * 需要以 root 权限运行以测试能力继承或用户切换

要添加这样的测试，通常需要在 `src/test` 中创建一个新的 C 程序。如果它可以独立运行并返回成功或失败，请将其添加到 `TESTS` 和 `noinst_PROGRAMS` 中。如果它需要被多次调用，或者需要被包装，请向 `TESTS` 添加新的 shell 脚本，并将新程序添加到 `noinst_PROGRAMS` 中。如果你需要访问 makefile 中的任何环境变量（例如用于 Python 解释器的 `${PYTHON}`），请确保 makefile 导出了它们。

## 使用 Stem 编写集成测试

'stem' 库包含针对 Tor 控制器协议的大量测试。你可以通过 `make test-stem` 从 tor 运行 stem 测试，或查看 `https://stem.torproject.org/faq.html#how-do-i-run-the-tests`。

要查看有哪些可用测试，请查看 stem 中的 `test/*` 目录。你首先注意到的是有 `unit` 和 `integ` 两类测试。前者是针对 stem 自身提供的功能的测试，可以单独测试，无需连接 tor 进程。这些关联性较小，除非你想开发新的 stem 功能。然而，后者是编写控制器功能测试的非常有用的工具。它们提供了一个带有已连接 tor 实例的默认环境，可以进行修改和查询。添加更多集成测试是提高 Tor 内部测试覆盖率的好方法，特别是对于控制器功能。

假设你确实想为一个之前未测试的控制器功能编写测试。我选择 `exit-policy/*` GETINFO 查询。由于这些是我们想要编写集成测试的控制器功能，要修改的正确文件是 `https://gitlab.torproject.org/tpo/network-health/stem/-/blob/master/test/integ/control/controller.py`。

首先我们注意到已经有一个名为 `test_get_exit_policy()` 的集成测试。这测试了 stem 的 `Controller.get_exit_policy()` 方法的交互，与我们的测试无关，因为没有 stem 方法可以使用所有 `exit-policy/*` 查询（如果有，可能已经测试过了。也许你想编写一个 stem 功能，但我选择只添加测试）。

我们的测试需要一个 tor 控制器连接，所以我们将对 `test_exit_policy()` 方法使用 `@require_controller` 注解。我们需要一个控制器实例，可以从 `test.runner.get_runner().get_tor_controller()` 获取。附带的 Tor 实例配置为客户端，但 exit-policy GETINFO 查询需要中继才能工作，所以我们必须更改配置（使用 `controller.set_options()`）。这对我们来说是可以的，我们只需记住设置 DisableNetwork 以便不真正启动出口中继，并且通过在测试结束时调用 `controller.reset_conf()` 来撤销我们所做的更改。此外，我们必须为 Tor 配置一个静态 Address，因为当它无法猜测合适的 IP 地址时，它会拒绝构建描述符。不幸的是，这类陷阱到处都是。如果你注意到任何看起来完全不合理的奇怪行为，不要忘记提交相应的工单。

查看上述文件中的 `test_exit_policy()` 函数以查看此测试的最终实现。

## 使用 Chutney 进行系统测试

'chutney' 程序在你的本地主机上配置并启动一组 Tor 中继、权威节点和客户端。它具有 `test network` 功能，可以通过它们发送流量并验证流量是否正确到达。

你可以通过将新测试添加到 `networks` 来编写新的测试网络。要将它们添加到 Tor 的测试中，请将它们添加到 `Makefile.am` 中的 `test-network` 或 `test-network-all` 目标。

（向 chutney 添加新类型的程序仍然需要修改代码。）

## 其他集成测试

使用 POSIX shell 调用 Tor 或测试系统的其他方面是完全可以的。当你这样做时，请查看我们在 `src/test/` 中的现有测试，确保你没有忘记任何重要的事项。例如：确保在各种构建场景中以正确的路径调用 Tor 可能会比较棘手。

我们在这里尽可能使用 POSIX shell，并使用 shellcheck 工具来确保我们的脚本可移植。我们只应为仅限开发人员使用的脚本要求 bash。
