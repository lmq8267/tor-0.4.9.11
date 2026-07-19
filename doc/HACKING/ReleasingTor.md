# 如何发布 Tor

以下是维护者在发布新 Tor 版本时应该采取的步骤。它分为 3 个阶段，并与我们的 Tor CI 发布流水线相关联。

在我们开始之前，首要规则是确保：

   - 我们的 CI（*nix 和 Windows）在每个要发布的版本上通过
   - Coverity 没有新的告警

## 0. 安全发布

首先，如果你正在进行安全发布，这必须在发布前几天完成：

   1. 如果这将是一个重要的安全发布，通过 `tor-packagers@lists.torproject.org` 给打包者提前警告。


## 1. 准备工作

以下事项必须在发布前**至少 2 天**完成：

   1. 在 dirauth-conf git 仓库中将版本添加为 RecommendedVersion 和 RequiredVersion，以便权威节点可以在发布前批准它们并将其纳入共识。

   2. 向 `tor-project@lists.torproject.org` 发送预发布公告，以通知 Tor 中的每个团队即将发布的版本。这是为了避免制造发布意外，并与其他团队同步。

   3. 请网络团队审查我们即将发布的所有版本中的 `changes/` 文件。此步骤是鼓励性的但不是强制性的。


## 2. 源码包

要构建要发布的源码包，我们需要启动 CI 发布流水线：

   https://gitlab.torproject.org/tpo/core/tor-ci-release

需要修改 `versions.yml` 以包含你要发布的 Tor 版本。完成后，git commit 并 push 以触发发布流水线。

前两个阶段（Preliminary 和 Patches）将自动运行。Build 阶段需要在所有生成的补丁合并到上游后手动触发。

   1. 从 `Patches` 阶段下载生成的补丁。

      将这些补丁应用到 `main` 或 `release` 分支上（视情况而定）。（版本号递增适用于 `maint`；任何涉及 changelog 的更改应仅适用于 `main` 或 `release`。）

      更新版本号时，将在 `maint` 分支上进行，因此要向前合并，请使用 `git merge -s ours`。例如，如果要将 `maint-0.4.5` 的版本变更合并到 `maint-0.4.6`，请在 `maint-0.4.6` 上执行此命令：`git merge -s ours maint-0.4.5`。然后你可以继续进行 git-merge-forward。

   2. 对于 ChangeLog 和 ReleaseNotes，你需要在顶部写一段简短说明，简要介绍本次发布。

   3. 审查、根据需要修改，然后将它们合并到上游。

   4. 手动触发 `Build` 阶段中的 `maintained` 作业，以便 CI 能够无错误地构建源码包。

完成后，每个选定的开发者需要使用以下工具以可重现的方式构建源码包：

   https://gitlab.torproject.org/tpo/core/tor-ci-reproducible

步骤如下：

   1. 运行 `./build.sh`，它将下载你需要的所有内容，包括发布 CI 中最新的源码包，并在校验和匹配时自动提交签名。你需要确认提交。

   2. 如果一切正常，`git push origin main` 推送你的签名。

所有选定开发者的签名都提交后：

   1. 手动触发 CI 发布流水线 `Post-process` 阶段中的 `signature` 作业。

   2. 如果通过，源码包和签名将作为构建产物可用，应该用于发布。

   3. 将它们放到 `dist.torproject.org`：

      将源码包及其签名上传到 dist 网站：

         `rsync -avP tor-*.gz{,.asc} dist-master.torproject.org:/srv/dist-master.torproject.org/htdocs/`

      然后，在 dist-master.torproject.org 上运行：

         `static-update-component dist.torproject.org`

      对于 alpha 版本或最新稳定版，在 https://gitlab.torproject.org/tpo/web/tpo 中提交一个 MR，更新 `databags/versions.ini` 以记录新版本。

      （注意：由于 #17805，一次只能列出一个稳定版本。但是，如果你的版本是稳定的，请不要称其为"alpha"，否则人们会感到困惑。）

      （注意：网站更新脚本需要一些时间来更新网站。）


## 3. 后续处理

一旦源码包已上传并准备就绪可以发布，我们需要执行以下操作：

   1. 使用 `git tag -s tor-0.x.y.z-<status>` 对版本进行打标签（`main` 分支或 `release` 分支，视情况而定），然后推送标签：`git push origin tor-0.x.y.z-<status>`

      （这应该是 `main` 或 `release` 分支，因为那是构建源码包的分支。我们希望我们的标签与源码包匹配。）

   2. 将 CI 发布流水线 `Post-process` 阶段中 `patches` 作业的构建产物合并到上游。

      与上面的步骤 (2.1) 一样，`-dev` 版本号递增需要用 `git merge -s ours` 手动完成。

   3. 在 `forum.torproject.net` 的 `News -> Tor Release Announcement` 分类中编写并发布发布公告。

      如果可以的话，请提及该发布将包含在哪个 Tor Browser 版本（附带日期）中。这通常仅适用于最新稳定版。

   4. 通过 `tor-announce@lists.torproject.org` 通知发布信息，指向论坛。在那里附上 ChangeLog。在我们可以直接从论坛自动化此类发布之前，我们会一直这样做。

   5. 通过向 https://gitlab.torproject.org/tpo/web/tpo 提交 MR 来更新 torproject.org 网站。

      `databags/versions.ini` 文件是需要更改为新发布版本的文件。

### 新稳定版

   1. 在版本标签处创建 `maint-x.y.z` 和 `release-x.y.z` 分支。然后用新版本更新 `./scripts/git/git-list-tor-branches.sh`。

   2. 在 `maint-x.y.z` 中用新版本更新 `./scripts/git/git-list-tor-branches.sh` 和 `./scripts/ci/ci-driver.sh`，然后向前合并到 main。（如果你还没有远程推送新分支，请合并本地分支。）

   3. 在 `main` 中，将版本号递增到下一个系列：`tor-x.y.0-alpha-dev`，然后打标签：`git tag -s tor-x.y.0-alpha-dev`

## 附录：通知打包者的替代方式

如果由于某种原因你需要联系一批打包者而不使用公开存档的 tor-packagers 列表，你可以尝试联系这些人：

       - {weasel,sysrqb,mikeperry} at torproject dot org
       - {blueness} at gentoo dot org
       - {paul} at invizbox dot io
       - {vincent} at invizbox dot com
       - {lfleischer} at archlinux dot org
       - {Nathan} at freitas dot net
       - {mike} at tig dot as
       - {tails-rm} at boum dot org
       - {simon} at sdeziel.info
       - {yuri} at freebsd.org
       - {mh+tor} at scrit.ch
       - {security} at brave.com
