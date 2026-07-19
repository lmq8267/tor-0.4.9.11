# Tor 的编码规范

tl;dr:

   - 使用 `--enable-fatal-warnings` 运行 configure
   - 为你的函数编写文档
   - 编写单元测试
   - 在提交补丁前运行 `make check`
   - 如果你更改了构建系统组件，请运行 `make distcheck`
   - 在 `changes` 目录中为你的分支添加一个文件。

## 补丁检查清单

如果可能的话，请将你的补丁以以下形式之一发送（按偏好降序排列）

   - 一个我们可以拉取的 git 分支
   - 由 git format-patch 生成的补丁
   - 一个统一差异

你是否记得...

   - 在使用 `--enable-fatal-warnings` 配置的情况下构建你的代码？
   - 运行 `make check-docs` 来查看所有新选项是否在手册页中？
   - 尽可能编写单元测试？
   - 运行 `make test-full` 来针对所有单元测试和集成测试进行测试（或者如果你有可用的互联网连接，运行 `make test-full-online`）？
   - 通过 `make distcheck` 测试发行版是否能正常工作？
   - 将你的代码基于适当的分支？
   - 在 `changes` 目录中适当地包含一个文件？

如果你正在提交一个主要的补丁或新功能，或者将来打算提交...

   - 设置 Chutney 和 Stem，参见 `doc/HACKING/WritingTests.md`
   - 运行 `make test-full` 来针对所有单元测试和集成测试进行测试。

如果你更改了构建系统组件：
   - 请运行 `make distcheck`
   - 例如，如果你更改了 Makefiles、autoconf 文件或任何其他影响构建系统的内容。

## 许可证问题

Tor 根据 LICENSE 中的许可条款进行分发——简而言之，即"三条款 BSD 许可证"。如果你向我们发送代码随 Tor 一起分发，它必须是我们可以在这些条款下分发的代码。请不要向我们发送补丁，除非你同意允许这样做。

一些兼容的许可证包括：

  - 三条款 BSD
  - 两条款 BSD
  - CC0 公共领域贡献

## 我们如何使用 Git 分支

每个主要开发系列（如 0.2.1、0.2.2 等）都有其主要工作应用于一个单一的分支。一次最多只能有一个系列是开发系列；所有其他系列都是维护系列，只接收 bug 修复。开发系列构建在一个名为 "main" 的 git 分支中；维护系列构建在名为 "maint-0.2.0"、"maint-0.2.1" 等的分支中。我们定期将活跃的 maint 分支向前合并。

对于除开发系列之外的所有系列，我们还有一个"发布"分支（如 "release-0.2.1"）。发布系列基于相应的维护系列，但它在大多数补丁上故意落后于 maint 系列，这样 bug 修复补丁通常不会被包含在维护发布中，直到它们在开发发布中经过了一段时间的测试。偶尔，我们会在将紧急 bug 修复合并到 maint 之前先合并到发布分支，但这很罕见。

如果你正在修复一个出现在特定版本中的 bug，请将你的 bug 修复分支基于存在该 bug 的第一个受支持系列的 "maint" 分支。（截至 2013 年 6 月，我们支持 0.2.3 及更高版本。）

如果你正在开发一个新功能，请将其基于 main 分支。如果你正在开发一个新功能，并且它需要一段时间来实现，和/或者你希望在实现功能时避免 Tor 中出现无关的 bug，请考虑从最新的 maint- 分支上分支。_永远_不要从 release- 分支分支。也不要从标签分支：它们来自发布分支。这样做很可能在将你的分支合并到 Tor 时产生噩梦般的合并冲突。最佳建议：不要尝试保持一个独立分支分叉超过 6 个月并期望它能干净地合并。尽早并经常地合并部分更改。

## 我们如何记录更改

当你做了一个需要 ChangeLog 条目的提交时，在 `changes` 顶层子目录中添加一个新文件。它的格式应该与当前 ChangeLog 文件中的单条目变更日志部分相同，例如

  o Major bugfixes (security):
    - Fix a potential buffer overflow. Fixes bug 99999; bugfix on
      0.3.1.4-beta.
  o Minor features (performance):
    - Make tor faster. Closes ticket 88888.

要编写一个 changes 文件，首先要对更改进行分类。一些常见的类别有：
  o Minor bugfixes (subheading):
  o Major bugfixes (subheading):
  o Minor features (subheading):
  o Major features (subheading):
  o Code simplifications and refactoring:
  o Testing:
  o Documentation:

subheading 是 Tor 中的一个特定领域。请参阅 ChangeLog 获取示例。

然后说明更改做了什么。如果是一个 bug 修复，请提到它修复了什么 bug 以及该 bug 是何时引入的。要找出该更改是在哪个 Git 标签中引入的，你可以使用 `git describe --contains <sha1 of commit>`。如果你不知道提交，你可以在 git 差异中搜索 (-S) 该功能的首次出现 (--reverse)。

例如，对于 #30224，我们想知道 bridge-distribution-request 功能是何时引入 Tor 的：

```console
$ git log -S bridge-distribution-request --reverse commit ebab521525
Author: Roger Dingledine <arma@torproject.org>
Date:   Sun Nov 13 02:39:16 2016 -0500

    Add new BridgeDistribution config option

$ git describe --contains ebab521525
tor-0.3.2.3-alpha~15^2~4
```

如果你需要知道包含某个提交的所有 Tor 版本，请使用：

```console
$ git tag --contains 9f2efd02a1 | sort -V
tor-0.2.5.16
tor-0.2.8.17
tor-0.2.9.14
tor-0.2.9.15
...
tor-0.3.0.13
tor-0.3.1.9
tor-0.3.1.10
...
```

如果一个 bug 是在当前最旧的受支持发布系列之前引入的，并且很难追踪它是在哪里引入的，你可以说"bugfix on all supported versions of Tor."

如果可能的话，尝试在进行更改的同一个提交中创建 changes 文件。请给它一个独特的名称，在你的更改的生命周期内没有其他分支会使用。我们通常使用 "ticketNNNNN" 或 "bugNNNNN"，其中 NNNNN 是工单编号。要验证 changes 文件的格式，你可以使用 `make check-changes`。它会作为 `make check` 的一部分自动运行——如果失败，我们必须尽快修复，以便我们的 CI 通过。这些检查在 `scripts/maint/lintChanges.py` 中实现。

changes 文件风格指南：
  * 保持一切简洁。

  * 从用户的角度来写：立即描述用户可见的更改。

  * 按名称提及配置选项。如果它们很少见或不寻常，请提醒人们它们的用途。

  * 用现在时和祈使语气描述更改：不要用过去时。

  * 每个 bug 修复都应该有一句类似 "Fixes bug 1234; bugfix on 0.1.2.3-alpha" 的话，描述修复了什么 bug 以及它的来源。

  * 使用 "Relays"，而不是"servers"、"nodes" 或 "Tor relays"。

当我们准备发布时，我们会将 changes 中的所有条目连接起来制作一个草案变更日志，并清空该目录。然后我们会将草案变更日志编辑成良好的可读格式。

什么需要 changes 文件？

   * 一个并非详尽的列表：任何可能改变用户可见行为的内容。任何足够改变内部实现、文档或构建系统以至于可能被人注意到的内容。大型或有趣的代码重写。任何可能在 6 个月后有人合理地想知道"那是什么时候发生的，和/或我们为什么那样做"的内容。

什么不需要 changes 文件？

   * 尚未在任何已发布版本的 Tor 中出现过的代码的 bug 修复
   * 对未包含在 tarball 中的文件的任何更改。这包括：
     * 对我们的 CI 配置的任何不影响分发源代码的更改。
     * 对仅开发者工具的任何更改，除非这些工具包含在 tarball 中。
   * 非功能性的代码移动。
   * 标识符重命名、注释编辑、拼写修复等。

为什么使用 changes 文件而不是 Git 提交信息？

   * Git 提交信息是为开发者写的，而不是为用户写的，而且事后几乎不可能修改。

为什么使用 changes 文件而不是 ChangeLog 中的条目？

   * 每个提交都修改 ChangeLog 文件倾向于产生大量的合并冲突。

## 空白和 C 语言一致性

Tor 的 C 代码按照 C99 标准编写。不时运行 `make check-spaces`，这样它就能告诉你与我们 C 语言空白风格的偏差。通常，我们使用：

   - Unix 风格的行尾
   - K&R 风格的缩进
   - 换行前不留空格
   - 绝不连续多个空行
   - 始终使用空格，绝不使用制表符
   - 每行不超过 79 列。
   - 每次缩进两个空格。
   - 控制关键字和相应括号之间留一个空格 `if (x)`、`while (x)` 和 `switch (x)`，绝不使用 `if(x)`、`while(x)` 或 `switch(x)`。
   - 任何内容和左花括号之间留一个空格。
   - 函数名和左括号之间不留空格。`puts(x)`，而不是 `puts (x)`。
   - 函数声明在行的开头。
   - 使用 `void foo(void)` 声明没有参数的函数。使用 `void foo()` 是 C++ 语法。
   - 新 API 使用 `const`。
   - 变量在声明时应该初始化，而不是在作用域顶部声明。

如果你使用的编辑器有 editorconfig.org 的插件，`.editorconfig` 文件将帮助你遵循这种编码风格。

我们努力在所有地方构建时都不产生警告。特别是，如果你使用 gcc，你应该使用 `--enable-fatal-warnings` 选项调用 configure 脚本。这将告诉编译器将所有警告变为错误。

## 应该使用的函数；不应该使用的函数

我们有一些包装函数，如 `tor_malloc`、`tor_free`、`tor_strdup` 和 `tor_gettimeofday;`，请使用它们而不是它们的通用等价物。（它们总是成功或退出。）

具体来说，不要使用 `malloc`、`realloc`、`calloc`、`free` 或 `strdup`。请使用 `tor_malloc`、`tor_realloc`、`tor_calloc`、`tor_free` 或 `tor_strdup`。

不要使用 `tor_realloc(x, y\*z)`。请使用 `tor_reallocarray(x, y, z)` 代替。

你可以通过查看 `src/lib/*/*.h` 获取 Tor 提供的兼容性函数的完整列表。你可以查看 `src/lib/containers/*.h` 中可用的容器。在编写太多代码之前，你可能应该熟悉这些模块，否则你最终会重新发明轮子。

我们不使用 `strcat` 或 `strcpy` 或 `sprintf` 等那些出了名的有问题的旧 C 函数。我们还避免使用 `strncat` 和 `strncpy`。请使用 `strlcat`、`strlcpy` 或 `tor_snprintf/tor_asprintf` 代替。

我们不直接调用 `memcmp()`。大多数情况下使用 `fast_memeq()`、`fast_memneq()`、`tor_memeq()` 或 `tor_memneq()`。如果你真的需要三态返回值，请使用 `tor_memcmp()` 或 `fast_memcmp()`。

不要直接调用 `assert()`。对于硬断言，使用 `tor_assert()`。对于软断言，使用 `tor_assert_nonfatal()` 或 `BUG()`。如果你需要在断言错误消息中打印调试信息，考虑使用 `tor_assertf()` 和 `tor_assertf_nonfatal()`。如果你编写的代码太底层而无法使用日志子系统，请使用 `raw_assert()`。

不要使用 `toupper()` 和 `tolower()` 函数。请使用 `TOR_TOUPPER` 和 `TOR_TOLOWER` 宏代替。类似地，使用 `TOR_ISALPHA`、`TOR_ISALNUM` 等代替 `isalpha()`、`isalnum()` 等。

当分配新字符串添加到 smartlist 时，使用 `smartlist_add_asprintf()` 一次完成两件事。

避免直接调用 BSD socket 函数。使用可移植的包装器来处理 socket 和 socket 地址。此外，socket 应该是 `tor_socket_t` 类型。

不要使用以下任何函数：它们不可移植。请使用以 `tor_` 为前缀的版本代替：strtok_r、memmem、memstr、asprintf、localtime_r、gmtime_r、inet_aton、inet_ntop、inet_pton、getpass、ntohll、htonll。（此列表不完整。）

## 什么代码可以使用什么其他代码？

我们正在努力随时间简化 Tor 的结构。从长远来看，我们希望 Tor 被构建成一组*没有循环依赖*的模块。

这个属性目前由 src/lib 中的模块提供，但在 Tor 的其余部分中没有。通常，较高层的库可以使用较低层的库，但永远不能反过来。

为了防止新的循环依赖产生，我们有一个工具，你可以通过 `make check-includes` 调用它，它也会作为 `make check` 的一部分自动运行。这个工具将验证对于每个带有 `.may_include` 文件的源代码目录，除了 `.may_include` 文件特别允许的本地头文件之外，没有包含其他本地头文件。在编辑这些文件之一时，请确保你没有在 Tor 的依赖图中引入任何循环。

## 浮点数学是困难的

计算机通常实现的浮点算术非常违反直觉。未能充分分析浮点的使用可能导致令人惊讶的行为，甚至安全漏洞！

一般建议：

   - 不要使用浮点。
   - 如果你必须使用浮点，请记录浮点精度和计算精度的限制如何影响函数输出。
   - 尽可能使用整数（可能作为定点数使用）进行计算，只在显示时转换为浮点。
   - 如果你必须在网络上发送浮点数，请以平台无关的方式序列化它们。Tor 避免交换浮点值，但如果交换，则使用 ASCII 数字，并使用小数点（"."）。
   - 二进制分数的行为与十进制分数非常不同。确保你理解这些差异如何影响你的计算。
   - 每次浮点算术运算都是丢失精度、溢出、下溢或产生其他不期望结果的机会。加法和减法往往比乘法和除法更糟（由于灾难性抵消等问题）。尝试安排你的计算以最小化这种影响。
   - 改变运算顺序会改变许多浮点计算的结果。当你简化计算时要小心！如果顺序很重要，请使用代码注释记录它。
   - 比较大多数浮点值的相等性是不可靠的。避免使用 `==`，改用 `>=` 或 `<=`。如果你使用 epsilon 值，请确保它适合所讨论的范围。
   - 不同的环境（包括编译器标志和单个平台上的每线程状态！）对相同的浮点计算可能得到不同的结果。这意味着你不能在需要确定性的内容中使用浮点数，例如共识生成。这也使得编写可靠的浮点输出单元测试变得困难。

有关更多有用的建议（以及一些背景知识），请参阅 [What Every Programmer Should Know About Floating-Point Arithmetic](https://floating-point-gui.de/)。

关于浮点算术的值得注意（和令人惊讶）的事实列表在 [Floating-point complexities](https://randomascii.wordpress.com/2012/04/05/floating-point-complexities/)。该[浮点系列文章](https://randomascii.wordpress.com/category/floating-point/)的大部分内容都很有帮助。

有关更详细（且数学密集）的背景知识，请参阅 [What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)。

## 其他 C 语言约定

`a ? b : c` 三元运算符只能在其他表达式内部使用；不要用它来替代 if。（在宏定义中必要时可以忽略此规则。）

赋值运算符不应嵌套在其他表达式内部。（在宏定义中必要时可以忽略此规则。）

## 二进制数据和线上格式

表示以 NUL 结尾的字符串时使用指向 `char` 的指针。表示任意二进制数据时使用指向 `uint8_t` 的指针。（许多旧的 Tor API 忽略此规则。）

不要试图通过将整数的指针转换为字节数组来编码整数。请使用类似 `set_uint32()`/`get_uint32()` 的函数，并且不要忘记字节序问题。

尽量不要手写新代码来解析或生成二进制格式。如果可能的话，请使用 trunnel。有关 trunnel 的更多信息，请参阅

    https://gitweb.torproject.org/trunnel.git/tree

有关向 Tor 添加新 trunnel 代码的信息，请参阅 src/trunnel/README

## 调用和命名约定

在可能的情况下，函数应在错误时返回 -1，在成功时返回 0。

对于多词标识符，使用小写单词加下划线组合。（例如 `multi_word_identifier`）。宏和常量使用全大写。

类型名应以 `_t` 结尾。

函数名应以模块名或对象名作为前缀。（通常，操作对象的代码应该是一个与对象同名的模块，所以很难分辨使用的是哪种约定。）

执行操作的函数应该使用祈使动词名称（例如 `buffer_clear`、`buffer_resize`）；返回布尔值的函数应该使用谓词名称（例如 `buffer_is_empty`、`buffer_needs_resizing`）。

如果你发现你有四个或更多可能的返回码值，可能是时候创建一个枚举了。如果你发现你正在向一个函数传递三个或更多标志，可能是时候创建一个接受位域的标志参数了。

## 优化什么

不要优化不在关键路径上的内容。目前，关键路径似乎是 AES、日志和网络本身。你可以自由地进行自己的性能分析来确定。

## 日志约定

[FAQ - Log Levels](https://www.torproject.org/docs/faq#LogLevel)

在正常的 OR 或 OP 操作期间不应期望出现任何错误或警告消息。

如果一个库函数当前被调用时失败总是意味着 ERR，那么库函数应该记录 WARN 并让调用者记录 ERR。

每个严重级别为 INFO 或更高的消息应该（A）对不了解 Tor 源代码的最终用户是可理解的；或者（B）以某种方式告知最终用户他们不期望理解该消息（也许使用类似 "internal error" 的字符串）。选项（A）优于选项（B）。

## Tor 中的断言

断言应该仅用于检测 bug。不要使用断言来检测错误的用户输入、网络错误、资源耗尽或类似问题。

Tor 始终在启用断言的情况下构建，因此请尝试仅在你绝对确定崩溃是最小坏选项的情况下使用 `tor_assert()`。许多 bug 是由于在应该使用更安全的检查时使用了 `tor_assert()` 造成的。

如果你正在编写一个断言来测试一个你_可以_从其恢复的 bug，请使用 `tor_assert_nonfatal()` 代替 `tor_assert()`。如果你想编写一个包含非致命断言的条件语句，请使用 `BUG()` 宏，如下所示：

```c
if (BUG(ptr == NULL))
	return -1;
```

## 分配器约定

按照约定，任何名称类似 `abc_t` 的 tor 类型都应该通过一个名为 `abc_new()` 的函数来分配。这个函数永远不应返回 NULL。

此外，名为 `abc_t` 的类型应该通过一个名为 `abc_free_()` 的函数来释放。不要直接调用这个 `abc_free_()` 函数——而是将其包装在一个名为 `abc_free()` 的宏中，使用 `FREE_AND_NULL` 宏：

```c
void abc_free_(abc_t *obj);
#define abc_free(obj) FREE_AND_NULL(abc_t, abc_free_, (obj))
```

这个宏将释放底层的 `abc_t` 对象，并将对象指针设置为 NULL。

你应该定义所有 `abc_free_()` 函数以接受 NULL 输入：

```c
void
abc_free_(abc_t *obj)
{
  if (!obj)
    return;
  tor_free(obj->name);
  thing_free(obj->thing);
  tor_free(obj);
}
```

如果你需要一个接受 `void *` 参数的释放函数（例如，用作函数回调），请使用类似 `abc_free_void()` 的名称定义它：

```c
static void
abc_free_void_(void *obj)
{
  abc_free_(obj);
}
```

在释放时，不要写类似 `if (x) tor_free(x)` 的代码。约定是当传入 NULL 指针时释放函数什么都不做。

## Doxygen 注释约定

使用一个或多个祈使句来描述函数的功能，就像你在告诉别人如何使用这个函数一样。换句话说，不要这样写：

```c
/** The strtol function parses a number.
 *
 * nptr -- the string to parse.  It can include whitespace.
 * endptr -- a string pointer to hold the first thing that is not part
 *    of the number, if present.
 * base -- the numeric base.
 * returns: the resulting number.
 */
long strtol(const char *nptr, char **nptr, int base);
```

相反，请这样写：

```c
/** Parse a number in radix <b>base</b> from the string <b>nptr</b>,
 * and return the result.  Skip all leading whitespace.  If
 * <b>endptr</b> is not NULL, set *<b>endptr</b> to the first character
 * after the number parsed.
 **/
long strtol(const char *nptr, char **nptr, int base);
```

Doxygen 注释是我们基于契约的抽象世界中的契约：如果调用你函数的函数依赖于它做某件事，那么你的函数应该在文档中提到它做了那件事。如果你依赖于一个函数做超出其文档描述的事情，那么你应该小心，因为它以后可能会做其他事情。
