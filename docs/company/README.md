# 公司定制版本维护

本目录仅属于公司发布分支，不提交到官方 Seafile PR。

## 分支职责

| 分支 | 用途 | 允许的更新 |
| --- | --- | --- |
| `main` | 官方 `main` 的同步镜像 | 仅同步官方提交 |
| `feat/seafile-datasource` | 向官方贡献 Seafile，[PR #3616](https://github.com/Tencent/WeKnora/pull/3616) | Seafile 改进、评审修复、合并官方 `main` |
| `company/release-0.8.2` | 基于 v0.8.2 的公司稳定维护线 | Seafile 修复、经过评估的官方修复、公司维护记录 |
| `company/release-<version>` | 新官方正式版的公司维护线 | 从新官方 tag 创建，移植仍需保留的自有补丁 |

公司发布分支不合并整个 PR 分支或官方 `main`，以免引入未纳入发布计划的改动。
公司分支上的通用 Seafile 修复应以独立提交回馈 PR；PR 中的新修复也以独立提交移植回来。
移植使用 `git cherry-pick -x <commit>` 记录来源，随后处理适配并安排回归。
不要将公司部署配置、密钥或此目录一起提交到官方 PR。

## 当前候选基线

以下为 2026-09-28 创建分支时的状态；完成验收和发布后更新此记录。

| 项目 | 值 |
| --- | --- |
| 官方版本 | `v0.8.2` |
| 官方提交 | `3e8b0bfc80b845b2d4b2ed683994748741450a97` |
| Seafile 候选代码提交 | `ee885721e257f6c09082d4bb2eb0f6884e801bd5` |
| 候选内容 | 官方 v0.8.2 + Seafile 后端、前端、已有测试和功能文档 |
| 建立分支后的新增内容 | 本维护文档；业务代码保持候选提交内容 |
| 数据库迁移 | Seafile 补丁未新增迁移 |
| 状态 | 待验收；本次分支整理没有运行测试或构建镜像 |
| 计划镜像标签 | `0.8.2-seafile.1`，尚未发布 |

### 补丁来源

| 提交 | 内容 |
| --- | --- |
| `d84373c89a242ba585839d35aa5f952c0ab181d7` | Seafile 后端连接器、增量游标、共享文件格式、入库接入和已有测试 |
| `cd6796d3c06c7b1e8874bb54754688ba2b8b9e65` | Seafile 前端、凭据输出适配、已有测试及使用文档 |
| `ee885721e257f6c09082d4bb2eb0f6884e801bd5` | 将上述功能与官方 v0.8.2 基线合并后的候选代码 |

最终补丁以 `git diff v0.8.2 company/release-0.8.2` 为准；不应仅根据提交名判断实际内容。
补丁台账后续追加官方修复链接、原提交、移植提交、适用版本和验收结果。

## 本地远程与日常同步

- `origin`：个人 fork，`git@github.com:Zaqqmm/WeKnora.git`，推送目标。
- `upstream`：官方仓库，`https://github.com/Tencent/WeKnora.git`，用于拉取。
- 本地配置了 `remote.pushDefault=origin`、`push.default=simple`、`pull.ff=only`。
- `main` 跟踪 `origin/main`；PR 和公司分支分别跟踪个人远程的同名分支。
- 上述远程与 Git 配置是本地设置，新克隆需自行配置。

同步官方 main（工作区干净时）：

```bash
git fetch upstream
git fetch origin
git switch main
git merge --ff-only upstream/main
git push origin main
```

如果快进合并或普通推送被拒绝，先检查本地、个人远程与官方的分歧，不强推覆盖。

更新 PR 分支：

```bash
git switch feat/seafile-datasource
git merge --ff-only origin/feat/seafile-datasource
git merge upstream/main
# 有冲突时逐项解决并提交；安排需要的回归。
git push origin feat/seafile-datasource
```

回到公司维护线：

```bash
git switch company/release-0.8.2
```

`pull.ff=only` 限制的是隐式 pull 合并；需要跟进官方的 PR 分支仍可显式执行上述 merge。
当前仓库是浅克隆，历史计数可能受浅边界影响；遇到缺少提交或无法确定共同祖先时，先从官方获取所需历史，再判断分歧。

## 发布规则

1. 公司 app 和 frontend 必须从同一个确定提交构建；源码 SHA 写入构建信息，并记录两个镜像的 digest。
2. 使用独立内部标签，如 `0.8.2-seafile.1`，不覆盖官方镜像标签。
3. Compose override 分别指定 app/frontend 镜像。DocReader 等其他服务继续固定已验证的配套版本；不要给所有服务套用只有 app/frontend 才有的内部标签。
4. Git 发布 tag 使用 `company-v0.8.2-seafile.1` 一类命名，在完成验收后创建，不移动已发布 tag。
5. 上游自带的 Docker 发布工作流匹配 `main` 和 `v*` tag。创建本公司分支不会触发该镜像发布工作流；公司构建发布流程需要单独实施。不要用官方的 `v*` 命名发布公司版本。
6. 上游现有的应用/前端 push 检查主要针对 `main`；公司分支的推送成功不代表自动完成测试。正式发布需要安排独立验收。

发布验收应覆盖：连接与目录选择、全量/增量同步、新增/修改/删除、失败重试和续传、其他数据源回归、凭据处理与 SSRF 配置。部署前保存配置及数据备份；升级涉及数据库迁移时，回退需同时考虑数据结构。

## 官方更新处理

- **新正式版**：从新 tag 创建新的公司发布分支，移植仍需保留的补丁并回归。旧维护线继续保留，部署验证后再切换。
- **紧急修复**：核对官方修复及其依赖，再移植到当前公司维护线，发布新的内部补丁版本。
- **官方合并 Seafile**：等包含该功能的正式版本发布后，比对 connector type、资源 ID、external ID、凭据和游标格式；验证现有数据兼容后移除自有补丁。
- **减少维护成本**：Seafile 逻辑优先留在连接器目录，共享框架只保留必要接入；每次升级审阅公司分支相对官方 tag 的最终差异。

目前只建立分支与维护记录，不代表已经启用自动跟进、自动发布或部署。
