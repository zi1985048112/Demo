# Notion

> 摘自《应用连接文档（AI 员工接入各应用指南）》第 25 章（完整版见 [../README.md](../README.md)）。状态图例：✅ 已实测可用；⚠️ 部分可用 / 有已知问题；❌ 当前不可用；⬜ 未测试。

Notion 是海外流行的在线笔记 + 知识库 + 协作工具，类似「飞书文档 + 飞书多维表格」合在一起：<b>页面（Page）</b>用来写文档、笔记、会议纪要、Wiki；<b>数据库（Database）</b>是表格形式的清单（任务、客户、项目进度等），可切换看板 / 日历视图。通过 **OpenMax Notion 连接器**接入，OAuth 授权，**直连模式（direct）**，共 25 个动作。实测：**除评论外均可用**（文件上传未测）。

## 能做什么

- 找和读：搜索页面和数据库、读页面内容、按条件查询数据库（筛选 / 排序）、列出工作区成员。
- 写：新建页面并写入内容（标题、段落、列表、待办等）、追加 / 修改 / 删除内容块、页面改名、删除页面（进回收站）。
- 数据库：新增一行、修改字段（如任务状态、截止日期）、新建 / 修改数据库。
- ❌ 评论：暂不能发评论、看评论（见踩坑第 1 条）。替代办法：在 Notion 网页 / App 里查看和回复评论；需要留言时，可让 AI 员工把内容直接写进页面正文。
- ⬜ 文件上传接口存在（3 个动作），未实测。

## 前置条件 / 所需权限 / 账号

- 一个 Notion 账号（notion.so / notion.com，邮箱或谷歌账号免费注册）。
- Notion 授权按页面放权：授权时选中的页面（及其子页面）agent 才能看到。建议先建一个测试页面，授权时选上它；需要 agent 管理哪些资料就选哪些。

## 详细接入步骤（已验证）

1. （建议）在 Notion 先建好要授权的页面 / 数据库。
2. OpenMax 工作台 →「应用连接」→ Notion → 登录 Notion，选择工作区，并**选择允许访问的页面** → 允许。
3. 授权给 AI 员工，机器人收到「连接已授权」通知（direct 模式）。
4. 机器人调 `notion-users/me` 和 `notion-search/search` 验证：能看到所选页面即成功。

## 测试清单

| 测试项 | 方法 | 实测结果 |
|---|---|---|
| 连接身份 / 成员列表 | notion-users/me、list | ✅ |
| 搜索页面和数据库 | notion-search/search | ✅ 能看到授权范围内的数据库和页面 |
| 读页面 / 页面内容 | notion-pages/get、notion-blocks/list-children | ✅ |
| 查询数据库（带筛选） | notion-data-sources/get、query | ✅ |
| 新建页面 + 写内容 + 读回 | notion-pages/create、notion-blocks/append-children | ✅ 标题 / 列表 / 待办读回一致 |
| 改内容 / 勾选待办 / 页面改名 | notion-blocks/update、notion-pages/update | ✅ |
| 数据库新增一行 + 改状态 | notion-pages/create（parent=data_source）、notion-pages/update | ✅ 新增一行并把状态改为「已完成」 |
| 删除内容块 / 删除页面 | notion-blocks/delete、notion-pages/update（in_trash） | ✅ 删除后进回收站 |
| 发评论 / 看评论 | notion-comments/create、list | ❌ 403「Insufficient permissions for this endpoint」（暂不支持） |
| 上传文件 | notion-file-uploads/\* | ⬜ 未测 |

## 踩坑记录与解决办法

- **评论报 403「Insufficient permissions for this endpoint」**：Notion 的评论权限（读评论 / 写评论）由平台侧的 Notion 集成应用配置决定，客户授权时无法自行添加，属于当前限制。客户问起口径：「Notion 连接目前不支持评论，读写页面和数据库都可以」。
- **搜不到某个页面**：授权时没选中该页面。解决：在 Notion 该页面右上角「···」→「连接」里添加 OpenMax 的连接，或在 OpenMax 重新授权并选上。
- **新版 Notion 接口的数据库**：数据库下面是「数据源（data source）」，查询 / 新增行用 data_source_id，而不是旧的 database_id。
- **删除进回收站**：删除页面是放入 Notion 回收站（30 天内可恢复），不是永久删除。

## 日常运维 / 注意事项

- 建议口径：读 / 搜索可直接做；新建页面、加数据库行先说明再做；删除、批量修改须客户明确同意。
- 客户问「连上 Notion 能干嘛」：找资料并总结、把会议纪要 / 日报直接写进 Notion、更新任务状态、往客户名单里加记录。
