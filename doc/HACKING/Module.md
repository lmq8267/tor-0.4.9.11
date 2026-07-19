# Tor 中的模块

本文档描述了在 Tor 中编写模块时的构建系统和编码标准。

## 什么是模块？

在 tor 代码库的上下文中，模块是一个可以在 `configure` 时选择性启用或禁用的子系统。

目前，tor 拥有以下模块：

  - Relay 子系统（relay）
  - 目录缓存系统（dircache）
  - 目录权威子系统（dirauth）

dirauth 代码位于其独立目录 `src/feature/dirauth/` 中。

relay 代码位于名为 `src/*/*relay` 的目录中，正在逐步重构和禁用。

dircache 代码位于 `src/*/*dircache` 中。目前，当且仅当 relay 模块被禁用时，它才会被禁用。（我们将它们视为独立模块，是因为它们在逻辑上相互独立，而不是因为你实际上会想要只运行其中一个而不用另一个。）

要禁用一个模块，请在 configure 时传入 `--disable-module-{dirauth,relay}`。目前所有模块默认都是启用的。

## 构建系统

构建系统的更改相当直接。

1. 在 `configure.ac` 文件中找到这个定义：`m4_define(MODULES`。它包含一个以空白字符分隔的 tor 中模块列表。将你的模块添加到该列表中。

2. 使用 `AC_ARG_ENABLE([module-relay]` 模板来创建你的新模块。我们采用"禁用模块"的方式，而不是逐个启用它们。因此，tor 默认会构建所有模块。

   这将定义 `HAVE_MODULE_<name>` 语句，可以在 C 代码中用于为你的模块进行条件编译。同时 `BUILD_MODULE_<name>` 也会为 automake 文件（例如：include.am）定义。

3. 在 `src/core/include.am` 文件中，找到 `MODULE_RELAY_SOURCES` 值。你需要为你的模块创建自己的 `_SOURCES` 变量，然后在应该构建该模块时有条件地将其添加到 `LIBTOR_A_SOURCES` 中。

   然后，将你的 SOURCES 变量添加到 `src_or_libtor_testing_a_SOURCES` 中以**非常重要**，这样测试才能构建它。

最后，你的模块将自动包含在 `TOR_MODULES_ALL_ENABLED` 变量中，该变量用于构建单元测试。它们始终构建所有内容以测试所有内容。

## 编码

如上所述，模块应隔离在其自己的目录中，位于 `src/*/` 下，以模块名称作为后缀。

有几条"规则"需要遵循：

* 尽可能减少进入模块的入口点数量。越少越好，当然这并不适用于所有用例。但是，始终牢记这一点是好事。

* **不要**在模块代码库之外使用 `HAVE_MODULE_<name>` 定义。每个入口点在模块被禁用时都应该有第二个定义。例如：

  ```c
  #ifdef HAVE_MODULE_DIRAUTH

  int sr_init(int save_to_disk);

  #else /* HAVE_MODULE_DIRAUTH */

  static inline int
  sr_init(int save_to_disk)
  {
    (void) save_to_disk;
    return 0;
  }

  #endif /* HAVE_MODULE_DIRAUTH */

  ```

  这种方式的主要原因是避免在代码库中到处都有条件代码。它应该尽可能集中，这有助于可维护性，同时避免了条件代码的混乱，使代码更容易跟踪和理解。

* 你的代码可能会有一部分需要被代码库的其余部分使用，但仍然是模块的一部分。一个很好的例子是，如果你查看 `src/feature/shared_random_client.c`：它包含了隐藏服务子系统所需的代码，但主要与 dirauth 模块中非常特定的共享随机子系统相关。

  这没问题，但尽量保持精简，并且永远不要使用与模块中相同的文件名。例如，这是一个坏主意，永远不应该这样做：

    - `src/feature/dirclient/shared_random.c`
    - `src/feature/dirauth/shared_random.c`

* 当你包含来自模块的头文件时，**始终**在语句中使用完整的模块路径。例如：

```c
#include "feature/dirauth/dirvote.h"`
```

  主要原因是我们**不会**默认添加模块的包含路径，因此需要指定它。同时，它也有助于我们的大脑理解哪个部分来自模块，哪个不是。

  即使在模块**内部**，也要像上面一样使用完整的包含路径。
