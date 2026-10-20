# Vercel

> 摘自《应用连接文档（AI 员工接入各应用指南）》第 27 章（完整版见 [../README.md](../README.md)）。状态图例：✅ 已实测可用；⚠️ 部分可用 / 有已知问题；❌ 当前不可用；⬜ 未测试。

Vercel 是海外流行的网站托管 / 上线平台，前端开发者常用：把网页代码或 GitHub 仓库交给它，几十秒就生成一个可访问的网址，还能绑定自定义域名，自带 HTTPS 和全球加速。可以类比国内的「云厂商静态网站托管 + CDN」，但用起来简单得多。通过 **OpenMax Vercel 连接器**接入，**API 令牌（api_key）方式，不是 OAuth 一键授权**，**直连模式（direct）**，共 44 个动作。实测：**实测 34/44 个动作，33 个通过**（10-19 补测删除部署、别名分配 / 删除、Edge Config 增删改、域名添加 / 移除）；新建令牌不可用（见踩坑）；购买域名、DNS 增删等高风险动作暂不测试。

## 能做什么

- 账号：当前账号信息 `vercel-account/me`、团队列表 `vercel-account/list-teams`、团队详情 `vercel-account/get-team`。
- 项目：列出 / 查看 / 新建 / 修改 / 删除 `vercel-projects/list` / `vercel-projects/get` / `vercel-projects/create` / `vercel-projects/update` / `vercel-projects/delete`。
- 部署（发布网站）：列出 / 查看 `vercel-deployments/list` / `vercel-deployments/get`；新建部署 `vercel-deployments/create`（可直接上传网页文件发布，也可从 Git 仓库发布）；部署事件 `vercel-deployments/get-events`、构建日志 `vercel-deployments/get-logs`；删除部署 `vercel-deployments/delete`。
- 环境变量：列出 / 查看 / 新建 / 修改 / 删除 `vercel-envs/list` / `vercel-envs/get` / `vercel-envs/create` / `vercel-envs/update` / `vercel-envs/delete`。
- 域名：账号下全部域名 `vercel-domains/list-all`、项目域名 `vercel-domains/list`、查看域名 / 域名配置 `vercel-domains/get` / `vercel-domains/get-config`、添加 / 移除 `vercel-domains/add` / `vercel-domains/remove`、购买域名 `vercel-domains/buy`。
- DNS：列出 / 新建 / 删除解析记录 `vercel-dns/list` / `vercel-dns/create` / `vercel-dns/delete`。
- 别名（给部署指定网址）：列出 / 分配 / 删除 `vercel-aliases/list` / `vercel-aliases/assign` / `vercel-aliases/delete`。
- Edge Config（在线配置存储）：列出 / 查看 / 读取条目 / 新建 / 更新条目 / 删除 `vercel-edge-config/list` / `vercel-edge-config/get` / `vercel-edge-config/get-items` / `vercel-edge-config/create` / `vercel-edge-config/update-items` / `vercel-edge-config/delete`。
- 令牌：列出 / 查看 / 新建 / 删除 `vercel-tokens/list` / `vercel-tokens/get` / `vercel-tokens/create` / `vercel-tokens/delete`。
- Webhook：列出 / 新建 `vercel-webhooks/list` / `vercel-webhooks/create`。
- ⚠️ **高风险动作（使用前须客户明确同意；删除项目 / 部署 / 别名 / Edge Config、移除域名已实测，其余暂不测试）**：购买域名 `vercel-domains/buy`（直接扣账户绑定的信用卡）；删除类 `vercel-projects/delete` / `vercel-deployments/delete` / `vercel-dns/delete` / `vercel-aliases/delete` / `vercel-domains/remove` / `vercel-edge-config/delete`（不可撤销）；删除令牌 `vercel-tokens/delete`（吊销正在用的令牌会让连接立即失效）。

## 前置条件 / 所需权限 / 账号

- 一个 Vercel 账号（vercel.com，用 GitHub 或邮箱免费注册，Hobby 免费版即可）；新账号自带一个默认团队。
- 令牌权限 = 令牌范围内账号能做的一切（包括删除和购买域名），所以令牌要妥善保管。

## 详细接入步骤（已验证）

1. 登录 Vercel → 右上角头像 → Account Settings → Tokens → Create：起个名字，Scope 选自己的账号 / 团队，Expiration 建议设有效期（测试时选 1 天或 7 天，不建议永不过期）；创建后复制令牌（**只显示一次**）。
2. OpenMax 工作台 →「应用连接」→ Vercel → 粘贴令牌并保存（direct 模式）。
3. 机器人调 `vercel-account/me` 验证：能读到用户名和默认团队即连接成功。

## 测试清单

| 测试项 | 方法 | 实测结果 |
|---|---|---|
| 账号信息 / 团队 | `vercel-account/me`、`vercel-account/list-teams` | ✅ |
| 项目：列出 / 新建 / 查看 / 修改 | `vercel-projects/list` / `vercel-projects/create` / `vercel-projects/get` / `vercel-projects/update` | ✅ 修改构建命令生效 |
| 发布网页（上传一个静态 index.html） | `vercel-deployments/create` | ✅ 几秒即 READY，自动分配正式网址 项目名.vercel.app，公网可打开；10-19 复测：不指定环境时直接发布到正式环境（见踩坑） |
| 查看部署 / 按项目列出部署 | `vercel-deployments/get` / `vercel-deployments/list` | ✅ |
| 部署事件 / 构建日志 | `vercel-deployments/get-events` / `vercel-deployments/get-logs` | ✅ 纯静态网页无构建，返回空，属正常 |
| 项目域名 / 别名 | `vercel-domains/list` / `vercel-aliases/list` | ✅ |
| 账号下全部域名 | `vercel-domains/list-all` | ✅ |
| 环境变量：新建 → 列出 → 修改 → 查看 → 删除 | `vercel-envs/create` → `vercel-envs/list` → `vercel-envs/update` → `vercel-envs/get` → `vercel-envs/delete` | ✅ |
| Edge Config / Webhook / 令牌列表（只读） | `vercel-edge-config/list` / `vercel-webhooks/list` / `vercel-tokens/list` | ✅ |
| 删除项目 | `vercel-projects/delete` | ✅ 返回 204，正式网址随即 404 |
| 删除部署 | `vercel-deployments/delete` | ✅（10-19）删除后查询返回 404 |
| 别名：分配 → 删除 | `vercel-aliases/assign` / `vercel-aliases/delete` | ✅（10-19）分配后网址公网可打开；删除后从别名列表消失 |
| Edge Config：新建 → 写入 / 修改 / 删除条目 → 删除 | `vercel-edge-config/create` / `vercel-edge-config/get` / `vercel-edge-config/update-items` / `vercel-edge-config/get-items` / `vercel-edge-config/delete` | ✅（10-19）每次写入后读回与预期一致；删除后查询返回 404 |
| 域名：添加 → 移除 | `vercel-domains/add` / `vercel-domains/remove` | ✅（10-19）添加测试域名后显示「待验证」（需按提示加 TXT 记录）；移除后从项目域名列表消失 |
| 新建令牌 | `vercel-tokens/create` | ❌（10-19）返回 403：用团队范围的令牌连接时无权新建令牌；该动作也不能指定团队（见踩坑） |
| 购买域名 / DNS 增删 / 删除令牌 / 新建 Webhook | `vercel-domains/buy`、`vercel-dns/create` / `vercel-dns/delete`、`vercel-tokens/delete`、`vercel-webhooks/create` | 暂不测试（高风险动作） |

## 踩坑记录与解决办法

- **默认开启「部署保护」（Vercel Authentication）**：每次部署的专属网址（带随机字符，如 项目名-xxxx-团队名.vercel.app）需要登录 Vercel 才能看，只有正式网址 项目名.vercel.app 是公开的。给客户 / 同事看时发正式网址，或在项目设置里关闭部署保护。
- **删项目后部署列表短暂残留**：`vercel-deployments/list` 里还会短暂出现该项目的部署记录，但 `vercel-deployments/get` 返回 404「Project not found」，属 Vercel 侧延迟，不用处理。
- **列表为空属正常**：新账号的域名、别名、Edge Config、Webhook 等列表类接口返回空是正常的。
- **令牌只显示一次**：丢了只能重新创建；令牌设为永不过期有泄露风险，建议设有效期并定期更换。
- **agent 侧注意**：`vercel-deployments/create` 直接上传文件（inline files）发布时，需同时传 name 和 project；项目不存在时会自动新建（实测为先建项目再部署）。
- **新建部署不指定环境，会直接发布到正式环境**（10-19）：`vercel-deployments/create` 不传 target 时，部署直接成为正式版本（production），返回时已是 READY，立即占用 项目名.vercel.app 等正式网址，公网可以打开。对已有项目操作时会直接替换线上版本，发布前须客户确认；只想预览时应指定预览环境（未实测）；测试用的部署 / 项目用完及时删除。
- **新建令牌报 403，无法通过 AI 员工创建**（10-19）：连接用的是团队范围的令牌，而 Vercel 只允许以个人账号身份新建令牌，所以返回 403；该动作又不允许传团队 ID（传了报 400「参数不合法」）。解决：新令牌请在 Vercel 网页 Account Settings → Tokens 里手动创建。

## 日常运维 / 注意事项

- 令牌过期或被删除后连接立即失效，需在 Vercel 重新创建令牌并在 OpenMax 工作台更新；建议定期轮换令牌。
- 发布到正式网址、绑定域名都是公开操作，先让客户确认；购买域名、各类删除操作须客户明确同意。
- 客户问「连上 Vercel 能干嘛」：把做好的网页 / 落地页直接发布成公网网址、查看部署状态和日志、管理环境变量和域名。
