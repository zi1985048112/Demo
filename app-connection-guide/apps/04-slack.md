# Slack

> 摘自《应用连接文档（AI 员工接入各应用指南）》第 4 章（完整版见 [../README.md](../README.md)）。状态图例：✅ 已实测可用；⚠️ 部分可用 / 有已知问题；❌ 当前不可用；⬜ 未测试。

Slack 是海外最流行的团队沟通软件，按工作区（Workspace）组织，有公开 / 私有频道、私聊和线程讨论，类似国内的「飞书 / 钉钉」的聊天部分。通过机器人端的 **Slack 渠道组件**接入：在 Slack 开发者后台用清单建 App，走 **Socket Mode 长连接**（不需要公网地址）。实测：**私聊、频道 @ 回复通过**，私聊可并入统一上下文；私有频道、smart 模式、文件收发未测。

## 能做什么

- 私聊 owner（可并入统一上下文）、在公开/私有频道里 @ 机器人对话。
- 线程感知回复、文件/图片收发、Markdown 格式、处理中表情提示。
- 频道 smart 模式（接收全部消息）或 mention 模式（只处理 @）。
- 在频道里让机器人帮忙 @ 某位同事（已实测）。
- ⚠️ **当前限制**：机器人不能修改用户的 Slack 界面语言等个人设置（需用户自己改）；私有频道、smart 模式、文件收发尚未实测。

## 前置条件 / 所需权限 / 账号

| 项目 | 说明 |
|---|---|
| Slack 工作区 | 需要有在该工作区安装 App 的权限（工作区管理员或被允许安装应用的成员） |
| Slack App | 在 api.slack.com/apps 创建，推荐“从清单（manifest）创建” |
| 两个 Token | **Bot Token**（xoxb- 开头）+ **App-Level Token**（xapp- 开头，需 `connections:write`）；Socket Mode 下**不需要** Signing Secret |
| 网络 | Socket Mode 由机器人主动连 Slack，不需要公网地址 |

## 详细接入步骤（已验证）

1. 在机器人端安装 Slack 渠道组件（直接告诉机器人“安装 Slack 组件”即可）。
2. 打开 api.slack.com/apps → **Create New App** → **From an app manifest** → 选择工作区 → 粘贴下面的清单（把 `MyAgent` 换成你的机器人名称）→ **Create**：

Slack App 清单（manifest）：

```yaml
display_information:
  name: MyAgent
features:
  bot_user:
    display_name: MyAgent
    always_online: true
oauth_config:
  scopes:
    bot:
      - app_mentions:read
      - chat:write
      - im:history
      - im:read
      - im:write
      - users:read
      - files:write
      - channels:history
      - groups:history
      - reactions:read
settings:
  event_subscriptions:
    bot_events:
      - app_mention
      - message.im
      - message.channels
      - message.groups
  socket_mode_enabled: true
```

> 💡 **补充建议（官方组件文档推荐，未实测）**：如需下载用户发来的文件、读取频道/私有频道信息、显示“处理中”表情，可在清单 scopes 中再加 `files:read`、`channels:read`、`groups:read`、`reactions:write`，修改后需重新安装 App。

3. 左侧 **Basic Information** → 页面下方 **App-Level Tokens** → **Generate Token and Scopes** → 填写 Token 名称 → **Add Scope** 选 `connections:write` → **Generate** → 复制 xapp- 开头的 Token。
4. 左侧 **Install App**（或 OAuth & Permissions）→ **Install to Workspace** → **Allow** → 复制 **Bot User OAuth Token**（xoxb- 开头）。
5. 把两个 Token **私聊**发给机器人完成配置（Token 只保存在机器人本机；不要发到群里、不要写进共享文档）。
6. **由 owner 本人**在 Slack 里私聊机器人第一条消息 → 自动绑定 owner（如发「你有哪些功能」，收到回复即打通）。
7. （可选）并入统一上下文。
8. 频道使用：在频道里 `/invite @机器人名`（或频道设置 → Integrations → Add apps）把机器人加进来 → 通知机器人管理端登记该频道 → 频道里 @ 机器人测试。私有频道需要 `groups:history`（清单已包含）。

（截图未收录）Basic Information 页：左侧菜单位置与 App Credentials 区域（Client Secret / Signing Secret 为隐藏状态，Socket Mode 下不需要）

（截图未收录）生成 App-Level Token：填写名称，Add Scope 选择 connections:write 后点 Generate

![App-Level Token 生成结果](../images/slack-app-level-token-created.png)

（截图未收录）OAuth & Permissions 页：复制 Bot User OAuth Token；修改权限后需点 Reinstall 重新安装

## 测试清单

| 测试项 | 方法 | 预期结果 | 实测结果 |
|---|---|---|---|
| 清单建 App + Socket Mode 连接 | 按上述接入步骤配置后启动 | 连接成功，无需公网地址 | ✅ |
| owner 自动绑定 + 私聊双向 | owner 首条私聊 | 绑定本人、正常回复 | ✅ |
| 并入统一上下文 | owner 同意 | 与其他渠道私聊共享上下文 | ✅ |
| 界面改中文 | 让机器人改 Slack 界面语言 | 机器人无法修改（个人设置） | ✅ 已确认机器人无法修改，需用户自己在 Preferences → Language & region 修改 |
| 频道 @ 回复（未登记频道，owner） | owner 在频道 @ 机器人 | owner 直通，正常回复 | ✅ |
| mention 模式下不 @ 不回复 | owner 在频道发不带 @ 的消息 | 不应回复 | ✅ 组件 0.1.2 及以上正常（0.1.1 会误回复，见踩坑第 4 条） |
| 让机器人在频道 @ 同事 | “帮我叫一下 @某人” | 机器人在频道 @ 该同事 | ✅（0.1.2 起 @显示名 正常保留） |
| 组件升级 | 升级组件 | 服务重启后正常 | ✅ 升级到 0.1.2、0.1.3 后均正常 |
| 非 owner 私聊被拒 | 另一账号私聊 | 自动拒绝 | ⬜ 未在 Slack 单独实测（同机制已在 Lark / 飞书渠道实测通过） |
| 私有频道 / smart 模式 / 文件收发 | — | — | ⬜ 未实测 |

## 踩坑记录与解决办法

1. **手动逐项勾权限容易漏**
   - 现象：机器人收不到某类消息。
   - 原因：scope 或事件订阅漏配。
   - 解决：用清单（manifest）建 App，一次配齐。
2. **App-Level Token 没加 `connections:write`**
   - 现象：Socket Mode 连不上。
   - 解决：重新生成 App-Level Token，并添加 `connections:write`。
3. **改了权限不生效**
   - 原因：Slack 要求修改 scope 后重新安装。
   - 解决：OAuth & Permissions → **Reinstall to Workspace**。
4. **mention 模式下 owner 不 @ 也被回复**
   - 原因：组件 0.1.1 的已知问题。
   - 解决：升级到 0.1.2 及以上（同时修复了 @显示名 被删除、同一条消息重复处理的问题）。
5. **Slack 界面语言改不了**
   - 原因：界面语言是个人账号设置，机器人无权修改。
   - 解决：用户自己在 Preferences → Language & region 修改。
6. **截图/文档里泄露 Token**
   - 现象：部署说明文档或截图里出现了完整可用的 Bot Token。
   - 风险：任何拿到文档的人都可以冒充机器人收发消息。
   - 解决：截图发布前必须打码；已泄露的 Token 应在 Slack 后台重新生成（轮换）并更新到机器人端。

## 日常运维 / 注意事项

- App ID 只是标识符，不是密钥；Bot Token / App-Level Token / Client Secret / Signing Secret 才是密钥。
- Socket Mode 偶尔出现心跳超时告警（“pong wasn't received…”，偶发），会自动重连，一般无需处理；若持续出现需检查网络。
- 组件 0.1.3 起：`groupPolicy=disabled` 对 owner 同样生效；频道默认触发方式为 mention。
- 机器人在频道里的回复所有成员可见，隐私内容请走私聊。
