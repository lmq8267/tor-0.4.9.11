# README.1st

## 本目录

本目录包含有关参与 Tor 开发所需了解的重要信息！

首先，阅读 `GettingStarted.md` 以了解如何开始 Tor 开发。

如果您决定编写补丁，`CodingStandards.md` 将为您提供大量关于我们代码组织方式的信息。

确保代码正确非常重要！阅读 `WritingTests.md` 将告诉您如何在 Tor 代码库中编写和运行测试。

我们使用许多其他程序来帮助维护和开发代码库：`HelpfulTools.md` 可以告诉您如何在 Tor 中使用它们。

如果您负责发布 Tor 版本，请参阅 `ReleasingTor.md` 以确保不会遗漏任何步骤！

## 补充信息

有关 Tor 应如何工作的完整信息，请参阅 [Tor 规范](https://gitlab.torproject.org/tpo/core/torspec) 中的文件。

有关如何修改 Tor 设计以实现不同工作的说明，请参阅 [Tor 提案流程](https://gitlab.torproject.org/tpo/core/torspec/-/blob/main/proposals/001-process.txt)。

要获取代码的最新版本，请安装 git，然后执行：

```console
$ git clone https://gitlab.torproject.org/tpo/core/tor.git
```

要获取 Tor 原始设计论文，请参阅 [此处](https://spec.torproject.org/tor-design)。请注意，自 2004 年以来 Tor 已在许多方面发生了变化。

有关大量安全论文（其中许多与 Tor 相关），请参阅 [Anonbib 的匿名精选论文](https://www.freehaven.net/anonbib/)。

## 保持联系

我们在 `tor-talk` 邮件列表上讨论 Tor。设计提案和讨论应发送到 `tor-dev` 邮件列表。我们在 irc.oftc.net 上活动，一般讨论在 `#tor` 频道，开发讨论在 `#tor-dev` 频道。

本 `HACKING` 目录中的其他文件在您开始使用 Tor 时也可能有用。

祝您编码愉快！
