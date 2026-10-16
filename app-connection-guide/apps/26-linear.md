# Linear

> 摘自《应用连接文档（AI 员工接入各应用指南）》第 26 章（完整版见 [../README.md](../README.md)）。状态图例：✅ 已实测可用；⚠️ 部分可用 / 有已知问题；❌ 当前不可用；⬜ 未测试。

Linear 是海外科技公司常用的项目管理 / 任务跟踪工具，主要给研发团队管需求和 bug，类似「飞书项目」、Jira、禅道、TAPD。基本概念：**团队（Team）**、**任务（Issue，含标题 / 描述 / 状态 / 负责人 / 优先级 / 标签）**、**项目 / 迭代（Project / Cycle）**。通过 **OpenMax Linear 连接器**接入，OAuth 授权，**直连模式（direct）**。实测：**读写全部通过**。

## 能做什么

- 连接器只有 2 个「万能」动作：`linear-graphql/query`（查询）和 `linear-graphql/mutate`（修改），agent 直接调用 Linear 官方 GraphQL 接口，Linear 网页上能做的基本都能做。
- 查：任务列表、按条件筛选（如「指派给我的、进行中的」）、团队 / 状态 / 项目 / 迭代 / 成员。
- 写：新建任务、改状态 / 负责人 / 优先级 / 标题 / 描述、新建和添加标签、评论（发 / 改 / 删）、删除任务。
- ✅ 实测未发现功能限制（删除任务会移入回收站，可恢复）。

## 前置条件 / 所需权限 / 账号

- 一个 Linear 账号（linear.app，邮箱或谷歌账号免费注册），注册时会建好工作区和团队。
- 授权账号在工作区里能做的事，agent 就能做（权限跟随账号）。

## 详细接入步骤（已验证）

1. OpenMax 工作台 →「应用连接」→ Linear → 登录 Linear，选择工作区 → 授权。
2. 授权给 AI 员工，机器人收到「连接已授权」通知（direct 模式）。
3. 机器人用查询动作查 `viewer`（当前账号）和 `teams` 验证。

## 测试清单

| 测试项 | 方法 | 实测结果 |
|---|---|---|
| 账号 / 工作区 / 团队 / 状态 | query viewer、organization、teams | ✅ |
| 任务列表 + 条件筛选 | query issues、viewer.assignedIssues(filter) | ✅ |
| 新建任务 | mutate issueCreate | ✅（标题 / 描述 / 优先级） |
| 改状态 / 负责人 / 优先级 / 标题 | mutate issueUpdate | ✅ |
| 新建标签并添加 | mutate issueLabelCreate、issueAddLabel | ✅ |
| 发 / 改 / 删评论 | mutate commentCreate / Update / Delete | ✅ |
| 删除任务 / 标签 | mutate issueDelete、issueLabelDelete | ✅ 删除后进回收站 |

## 踩坑记录与解决办法

- 实测未遇到功能限制。
- **新建任务默认进 Backlog**：不指定状态时进入「Backlog」，需要直接进「Todo / 进行中」时要带上状态 ID（先查团队的 states）。
- **删除进回收站**：issueDelete 是移入回收站（可恢复），不是永久删除。
- **agent 侧注意**：两个动作都要传完整的 GraphQL 语句；查询用 query，任何修改必须用 mutate。

## 日常运维 / 注意事项

- 建议口径：查任务可直接做；建任务、改状态 / 指派先说明再做；删除须客户明确同意。
- 客户问「连上 Linear 能干嘛」：查我手上还有哪些没做完、这周迭代进度、把客户反馈的 bug 直接建成任务并指派。
