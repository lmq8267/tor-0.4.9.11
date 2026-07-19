# 电路子系统追踪事件

电路子系统会发出一系列与电路对象生命周期及其状态变化相关的追踪事件。

本文档描述了每个事件记录的数据及其含义。

## 背景

电路有两种类型：origin（源）电路和 OR（洋葱路由器）电路。两者都派生自一个称为通用电路的基础对象。

- Origin 电路是由 tor 自身发起的电路，例如客户端或洋葱服务电路。

- OR 电路是经过我们但不是由我们发起的电路，因此只有中继节点能看到。

许多操作在基础（通用）电路上执行，有些则特定于 origin 或 OR 电路。以下章节按电路类型对它们逐一进行描述。

## 追踪事件

对于 LTTng tracer，这些事件的子系统名称为：`tor_circuit`。

另外，除非另有说明，每个事件都会发出一组通用参数，因此应始终按以下顺序预期它们：

- `circ_id`：对于 origin 电路，这是在 cell 中使用的全局电路标识符。对于 OR 电路，该值为 0。

- `purpose`：电路的用途，即它的使用目的。请注意，这在电路的生命周期中可能会发生变化。可能值的完整列表请参见 `core/or/circuitlist.h` 中的 `CIRCUIT_PURPOSE_*`。

- `state`：电路的状态。这在电路的生命周期中会发生变化。可能值的完整列表请参见 `core/or/circuitlist.h` 中的 `CIRCUIT_STATE_*`。

现在来看追踪事件。

### 通用电路 (`circuit_t`)

以下事件针对基础电路对象触发，因此适用于所有类型的电路。

  * `free`：电路对象被释放，即内存已释放且不再可用。在此事件之后，不会再为该特定电路对象发出更多事件。

  * `mark_for_close`：电路对象被标记为关闭，即安排在后续的主循环周期事件中关闭。

    额外参数：

    - `end_reason`：电路关闭的原因。Tor 终端经常会将该原因更改为某些通用内容，以避免向端点泄露内部原因。因此，该值可能与 orig_close_reason 不同。

    - `orig_close_reason`：电路关闭的原始原因。该值永远不会改变，包含我们关闭它的内部原因。**永远**不会将这个原因通过电路回传。

  * `change_purpose`：用途变更。

    额外参数：

    （不包含 `purpose` 参数）

    - `old_purpose`：之前的用途，不再使用。

    - `new_purpose`：分配给电路的新用途。

  * `change_state`：状态变更。

    额外参数：

    （不包含 `state` 参数）

    - `old_state`：之前的状态，不再使用。

    - `new_state`：分配给电路的新状态。

### Origin 电路 (`origin_circuit_t`)

以下事件仅针对 origin 电路触发。

  * `new_origin`：新的 origin 电路已被创建，意味着它已被新分配、初始化并添加到全局列表中。

  * `establish`：电路正在建立中。这是初始的第一步，路径已被选择，并且与第一跳的连接已启动。

  * `cannibalized`：电路已被征用。当我们有一个已打开的未使用电路（抢先创建的电路）并被选中时，就会发生这种情况。

  * `first_onion_skin`：已发送第一个 onion skin，即与第一跳的握手。

    额外参数：

    - `fingerprint`：第一跳的身份摘要（RSA）。

  * `intermediate_onion_skin`：已发送中间 onion skin，这可能是第一跳之后的任何一跳。因此会有 `N - 1` 个此类事件，其中 `N` 是路径中的总跳数。

    额外参数：

    - `fingerprint`：下一跳的身份摘要（RSA）。

  * `opened`：电路刚刚变为打开状态，意味着路径上的所有跳都已与我们完成了握手协商，电路现在可以发送 cell 了。

  * `timeout`：电路已超时，即我们等待电路建立的时间过长。

  * `idle_timeout`：电路因空闲而超时。这由 MaxCircuitDirtiness 参数控制，默认为 10 分钟。

对于常见的 3 跳电路用例，应按以下顺序看到以下事件：

  `new_origin` -> `establish` -> `first_onion_skin` ->
  `intermediate_onion_skin` -> `intermediate_onion_skin` -> `opened`

### OR 电路 (`or_circuit_t`)

以下事件仅针对 OR 电路触发。对于每个事件，`circ_id` 参数不存在，因为它始终为 0。`purpose` 和 `state` 参数保留。

  * `new_or`：新的 OR 电路已被创建，意味着它已被新分配、初始化并添加到全局列表中。
