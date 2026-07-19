# 仓库指南

Tor（`tor-0.4.9.11`）的贡献者指南。另请参阅 `doc/HACKING/` 目录中的开发文档。

## 项目结构与模块组织

Tor 是一个使用 GNU Autotools（`configure.ac`、`Makefile.am`）构建的 C 语言守护进程。

- `src/lib` — 底层工具库：容器、加密、缓冲区、压缩、编码、时间、调度。
- `src/core` — 核心进程内逻辑：主循环、OR 协议、加密子系统。
- `src/feature` — 功能子系统：HS、中继、目录权威/目录缓存/目录客户端、控制、节点列表、度量。
- `src/app` — 应用入口点、配置和 `tor` 二进制文件的组装。
- `src/ext` — 引入的第三方代码（如 ed25519、curve25519-donna）。
- `src/trunnel` — 从 `.trunnel` 定义生成的线路格式解析器。
- `src/test` — C 单元测试、集成测试、模糊测试目标和 Python 辅助工具。
- `doc/HACKING` — 开发者文档（`CodingStandards.md`、`WritingTests.md`、`Module.md`）。
- `changes/` — 每个变更日志条目一个文件（见下文）；在发布时清空。
- `contrib/` 和 `scripts/` — 运维工具和维护者脚本。

## 构建、测试和开发命令

从发布压缩包构建：
```sh
./configure && make && make install      # 标准构建
```
从全新克隆的 git 仓库构建时，需先运行 `./autogen.sh`。

常用目标（在构建根目录下运行）：
- `make check` — 运行捆绑的单元测试和 lint 检查（`check-spaces`、`check-changes`）。
- `make test-full` — `check` 加上 Stem 和 Chutney 集成测试。
- `make test-full-online` — 另外运行需要网络访问的测试。
- `./src/test/test <prefix>/..` — 运行部分单元测试；`--no-fork` 禁用子进程隔离；`--debug` 启用详细日志。
- `make check-spaces` — 强制执行空白/缩进规则。
- `make distcheck` — 验证源代码树是否正确打包（构建系统修改后必须运行）。
- `scripts/coccinelle/apply.sh` — 应用项目使用的 Coccinelle 重构。

## 编码风格与命名约定

Tor 遵循 C99 标准、K&R 缩进（两个空格）、Unix 换行符、不使用制表符、不超过 79 列，以及 `if (x)`/`func(y)` 的间距风格。项目提供了 `.editorconfig` 文件。完整规则请参阅 `doc/HACKING/CodingStandards.md`。

- 使用 Tor 封装函数（`tor_malloc`、`tor_free`、`tor_strdup`、`tor_reallocarray`）代替 libc 的等效函数。
- 新 API 使用 `const`；函数声明使用 `void foo(void)` 而不是 `void foo()`。
- 在声明处初始化变量，而不是在作用域顶部。
- 模块（如 `relay`、`dirauth`、`dircache`）在 configure 时通过 `--disable-module-{name}` 进行开关控制；参见 `doc/HACKING/Module.md`。
- 优先使用 `scripts/maint/` 中的 linter（CheckSpace、checkIncludes、practracker）在本地捕获问题。

## 测试指南

- 单元测试位于 `src/test/test_*.c`；编译后的运行程序为 `./src/test/test`，慢速测试在 `./src/test/test-slow`。
- 模糊测试目标位于 `src/test/fuzz/`（运行 `make check` 执行静态模糊测试）。
- 集成测试使用 Stem（`STEM_SOURCE_DIR` + `make test-stem`）和 Chutney（`CHUTNEY_PATH` + `make test-network`）。
- 覆盖率：使用 `--enable-coverage` 配置，运行测试，然后执行 `./scripts/test/coverage <tmpdir>`。
- 每个非平凡的更改都应在可行的情况下包含单元测试。

## 提交和拉取请求指南

- 错误修复基于 `maint-*` 分支；功能开发基于 `main`。切勿从 `release-*` 或标签分支。
- 以 git 分支（最佳）、`git format-patch` 或统一差异格式发送补丁。
- 构建前使用 `--enable-fatal-warnings` 配置；确保 `make check` 通过。
- 在 `changes/` 下添加变更日志文件，命名为 `ticketNNNNN` 或 `bugNNNNN`。格式由 `make check-changes`（`scripts/maint/lintChanges.py`）验证：
  ```
  o Minor bugfixes (subsystem):
    - Describe the change. Fixes bug 1234; bugfix on 0.3.5.1-alpha.
  ```
- 使用现在时态、祈使语气；使用"Relays"，而非"servers"或"nodes"。
- 仅接受兼容的许可证（优先采用 3-clause BSD）；参见 `CONTRIBUTING` 和 `LICENSE`。
