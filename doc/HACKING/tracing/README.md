# Tracing

本文档描述了 tor 中事件追踪子系统的工作原理，以便开发者能够向代码库中添加事件，并将其挂接到事件追踪框架（即 tracer）上。

## 警告 ##

在生产环境（公共网络）中使用时，追踪 tor 守护进程**始终**会产生敏感数据。

研究人员使用追踪来研究自己的 tor 客户端（例如：构建路径、计时或性能）是**合乎道德的**。

将包含其他用户活动（例如中继数据或任何处理用户流量的数据）的数据进行归档、发布或保留是**不道德的**。这当然也包括任何 notice 级别以下的日志。

发布包含用户流量的追踪数据分析结果也是**不安全的**。

换句话说，包含其他用户活动的追踪数据**不**可以以任何形式安全发布。

## 基础 ###

追踪分为两个不同的概念：追踪 API 和追踪探针。

API 位于 `src/lib/trace/` 中，定义了如何在 tor 代码中调用跟踪点。任何 C 文件如果想调用跟踪点，都应该包含 `src/lib/trace/events.h`。

探针是实际记录跟踪点数据的部分。因为它们通常需要访问特定的子系统对象，所以探针位于每个子系统内部。它们定义在 `trace-probes-<subsystem>.c` 文件中。

### 事件

一个追踪事件本质上是一个函数，我们可以通过它传递任何想要收集的数据。此外，我们还为事件指定了上下文，例如子系统名称和事件名称。

tor 中的追踪事件具有以下标准格式：

```c
tor_trace(subsystem, event_name, args...);
```

`subsystem` 参数是追踪事件所属子系统的名称。例如可以是 "scheduler"、"vote" 或 "hs"。其目的是为事件添加一些上下文，以便我们在收集数据时知道数据来自何处。

`event_name` 是事件的名称，为事件添加更好的语义。

`args` 可以是我们想要收集的任意数量的参数。

以下是在 main() 中可能的跟踪点示例：

```c
tor_trace(main, init_phase, argc);
```

上述代码是在 `main` 子系统中的一个跟踪点，事件名称为 `init_phase`，`int argc` 作为参数传递给事件。

`argc` 如何被收集或使用与插桩（向代码中添加追踪事件）无关。这是 tracer 的工作，这就是为什么追踪事件和收集框架（tracer）是解耦的。即使没有 tracer，你_也可以_拥有追踪事件。

### 插桩 ###

在 `src/lib/trace/events.h` 中，我们将高级的 `tor_trace()` 宏映射到一个或多个已启用的插桩。

目前，我们有 3 种可能的插桩类型：

1. Debug

  这会将每个跟踪点映射到 `log_debug()`。但是，不会传递任何参数，因为我们不知道它们的类型也不知道调试日志的字符串格式。输出标准化如下：

```
[debug] __FUNC__: Tracepoint <event_name> from subsystem <subsystem> hit.
```

2. USDT

  用户静态定义追踪（USDT）是一种探针，可以被多种 tracer 处理，例如 SystemTap、DTrace、perf、eBPF 和 ftrace。

  对于每种 tracer，需要定义 ABI 以便 tracer 能够从跟踪点对象中提取数据。例如，tracer 需要知道如何打印 `circuit_t` 对象的电路状态。

3. LTTng-UST

  LTTng 用户空间是一种具有自己插桩类型的 tracer。探针定义在 C 代码中创建，且是强类型的。

  更多信息请参见 https://lttng.org/docs。

## 构建系统

本节描述了插桩如何集成到 tor 的构建系统中。

默认情况下，tor 中的所有追踪事件都是禁用的，即 `tor_trace()` 是一个空操作（NOP），因此没有执行时间开销。

要启用特定的插桩，有以下配置选项：

1. Debug: `--enable-tracing-instrumentation-debug`

2. USDT: `--enable-tracing-instrumentation-usdt`

3. LTTng: `--enable-tracing-instrumentation-lttng`

它们可以一起使用，也可以独立使用。如果设置了其中任何一个，`HAVE_TRACING` 宏就会被定义。对于每种插桩类型，还会分别设置 `USE_TRACING_INSTRUMENTATION_<type>` 宏。

## 添加跟踪点 ##

这非常简单。假设你想在 `src/feature/rend/rendcache.c` 中添加一个追踪事件，你首先需要包含以下文件：

```c
#include "lib/trace/events.h"
```

然后，`tor_trace()` 宏可以按照前面章节中详述的特定格式使用。例如：

```c
tor_trace(hs, store_desc_as_client, desc, desc_id);
```

对于 `Debug` 插桩，你不需要做其他任何事情。

对于 `USDT` 插桩，你需要以特定 tracer 能理解的方式定义探针。例如，SystemTap 要求你为每个跟踪点定义一个 `tapset`。

对于 `LTTng`，你需要在 `trace-probes-<subsystem>.{c|h}` 文件中定义探针。请参阅 `trace-probes-circuit.{c|h}` 文件作为示例，以及 https://lttng.org/docs/v2.11/#doc-instrumenting。

## 性能 ##

关于启用跟踪点时的性能说明。跟踪点（USDT、LTTng-UST 等）的目标之一是它们可以被启用或禁用。默认情况下，它们是禁用的，这意味着 tracer 不会记录数据，但它必须进行一次检查，因此开销基本上就是一次 `分支` 的代价。

如果启用，则性能取决于 tracer。对于 LTTng-UST，每个事件的开销约为 110 纳秒。
