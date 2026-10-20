# Slack

> 摘自《应用连接文档（AI 员工接入各应用指南）》第 4 章（完整版见 [../README.md](../README.md)）。状态图例：✅ 已实测可用；⚠️ 部分可用 / 有已知问题；❌ 当前不可用；⬜ 未测试。

Slack 是海外最流行的团队沟通软件，按工作区（Workspace）组织，有公开 / 私有频道、私聊和线程讨论，类似国内的「飞书 / 钉钉」的聊天部分。通过机器人端的 **Slack 渠道组件**接入：在 Slack 开发者后台用清单建 App，走 **Socket Mode 长连接**（不需要公网地址）。实测（10-19 补测）：**私聊、公开频道 / 私有频道 @ 回复、smart 模式均通过**，私聊可并入统一上下文；**图片 / 文件收发需要应用开通 files 权限；收文件依赖组件修复（已提研发）**。

## 能做什么

- 私聊 owner（可并入统一上下文）、在公开 / 私有频道里 @ 机器人对话（私有频道要先把机器人邀请进频道，10-19 实测通过）。
- 线程感知回复、Markdown 格式、处理中表情提示；发图片 / 文件（需开通 `files:read`、`files:write` 权限），接收图片、PDF 等文件（依赖组件修复，见踩坑第 8 条）。
- 频道 smart 模式（不 @ 也接收，机器人自己判断：有人提问会回复，闲聊不插话，10-19 实测）或 mention 模式（只处理 @）；可统一设置，也可按频道单独设置，改完即时生效、无需重启。
- 在频道里让机器人帮忙 @ 某位同事（已实测）。
- ⚠️ **当前限制**：机器人不能修改用户的 Slack 界面语言等个人设置（需用户自己改）；**图片 / 文件收发需要应用开通 files 权限**（缺少时发送失败，见踩坑第 7 条）；**收文件依赖组件修复（已提研发）**，组件升级后需复查；中文文件名保存后会变成下划线（内容不受影响）。

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
      - files:read
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

> 💡 **文件权限（10-19 实测）**：机器人收发图片 / 文件需要 `files:read` 和 `files:write` 两个权限（上面清单已包含）；缺少时机器人发文件会报 missing_scope（见踩坑第 7 条）。**补充建议（官方组件文档推荐，未实测）**：如需读取频道 / 私有频道信息、显示“处理中”表情，可再加 `channels:read`、`groups:read`、`reactions:write`。修改 scope 后都需要在 OAuth & Permissions 页点 **Reinstall to Workspace** 重新安装。

3. 左侧 **Basic Information** → 页面下方 **App-Level Tokens** → **Generate Token and Scopes** → 填写 Token 名称 → **Add Scope** 选 `connections:write` → **Generate** → 复制 xapp- 开头的 Token。
4. 左侧 **Install App**（或 OAuth & Permissions）→ **Install to Workspace** → **Allow** → 复制 **Bot User OAuth Token**（xoxb- 开头）。
5. 把两个 Token **私聊**发给机器人完成配置（Token 只保存在机器人本机；不要发到群里、不要写进共享文档）。
6. **由 owner 本人**在 Slack 里私聊机器人第一条消息 → 自动绑定 owner（如发「你有哪些功能」，收到回复即打通）。
7. （可选）并入统一上下文。
8. 频道使用：在频道里 `/invite @机器人名`（或频道设置 → Integrations → Add apps）把机器人加进来（**私有频道必须先邀请**，否则机器人收不到消息）→ 通知机器人管理端登记该频道（owner 本人在未登记的频道里 @ 也能直接用；其他成员要用必须先登记）→ 频道里 @ 机器人测试。私有频道需要 `groups:history`（清单已包含）。

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
| 非 owner 私聊被拒 | 另一账号私聊 | 自动拒绝 | 暂不测试（同机制已在 Lark / 飞书渠道实测通过） |
| 私有频道 @ 回复（未登记频道，owner） | 先 /invite 机器人进私有频道，owner 在频道里 @ 机器人 | 收到并回复 | ✅ owner 直通（不受 groupPolicy=allowlist 限制）；其他成员需管理端先登记该频道（10-19） |
| smart 模式 | 频道设为 smart，不 @ 分别发提问和闲聊 | 提问会回复，闲聊不插话 | ✅ 「明天几点开会？」得到回复，「今天天气不错」未插话；回复能结合频道里之前的消息；切换模式即时生效、无需重启（10-19） |
| 机器人发送图片 / 文件 | 机器人在私聊回发图片、txt | 用户能看到并打开 | ✅ 开通 files:read、files:write 并重新安装 App 后正常，图片和 txt 都能下载打开；未开通时报 missing_scope（见踩坑第 7 条）（10-19） |
| 接收图片 / 文件 | owner 给机器人发截图、11 页 PDF | 机器人能下载并读取内容 | ✅ 组件修复后均可读取（原组件会把带附件的消息直接丢掉，见踩坑第 8 条）；中文文件名会变成下划线，不影响内容（10-19） |

## 踩坑记录与解决办法

1. **手动逐项勾权限容易漏**
   - 现象：机器人收不到某类消息。
   - 原因：scope 或事件订阅漏配。
   - 解决：用清单（manifest）建 App，一次配齐。
2. **App-Level Token 没加 `connections:write`**
   - 现象：Socket Mode 连不上。
   - 解决：重新生成 App-Level Token，并添加 `connections:write`。
3. **改了权限不生效**
   - 现象：已在后台加了 scope（如 `files:read` / `files:write`），机器人仍报 missing_scope。
   - 原因：Slack 要求修改 scope 后重新安装 App，新权限才生效。
   - 解决：OAuth & Permissions → **Reinstall to Workspace**。10-19 实测：重新安装后 Bot Token 不变，机器人端无需改配置。
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
7. **机器人发图片 / 文件失败（missing_scope）**
   - 现象：机器人回发图片、文件失败，报 missing_scope；文字消息正常。
   - 原因：Slack App 缺少 `files:read` / `files:write` 权限（我们的实例一开始就缺这两个）。
   - 解决：api.slack.com/apps → 选中该 App → OAuth & Permissions → Bot Token Scopes → 添加 `files:read`、`files:write` → 点 **Reinstall to Workspace**。重新安装后 Bot Token 不变，无需改机器人配置（10-19 实测，之后图片和 txt 都能在私聊里收到并打开）。
8. **用户发的图片 / 文件机器人没反应**
   - 现象：给机器人发图片、PDF（带不带文字都一样）没有任何反应，纯文字消息正常。
   - 原因：Slack 里带附件的消息属于 file_share 类型；组件原来会把所有带类型标记的消息（编辑、入群通知等）一律丢掉，附件消息也被悄悄丢弃。
   - 解决：已在我们的实例上做了本地修复（组件消息过滤处放行 file_share，src/index.js），之后截图和 11 页 PDF 都能收到并读取（10-19）；已提研发（《应用连接器产品问题清单（转研发）》第 8 条）。**研发合并修复前，组件升级可能覆盖本地修复，每次升级后要复测一次收文件**。对客户口径：「图片 / 文件收发需要应用开通 files 权限；收文件依赖组件修复（已提研发）」。
9. **私有频道里 @ 机器人没反应**
   - 现象：在私有频道 @ 机器人，没有回复。
   - 原因：机器人不是该私有频道成员，收不到频道消息；或者发消息的不是 owner，而频道还没登记（groupPolicy=allowlist 时只响应登记过的频道，owner 除外）。
   - 解决：先在频道里 `/invite @机器人名`；非 owner 要用时由管理端登记频道：`node ~/zylos/.claude/skills/slack/src/admin.js add-group <频道ID> <名称>`。
10. **中文文件名变成下划线**
    - 现象：机器人收到的文件，保存后中文文件名变成一串下划线，文件内容完整。
    - 原因：组件保存文件时会把中文等字符替换成下划线（Teams 渠道同样如此）。
    - 解决：不影响阅读内容；需要按文件名区分时，在消息里补一句说明。

## 日常运维 / 注意事项

- App ID 只是标识符，不是密钥；Bot Token / App-Level Token / Client Secret / Signing Secret 才是密钥。
- Socket Mode 偶尔出现心跳超时告警（“pong wasn't received…”，偶发），会自动重连，一般无需处理；若持续出现需检查网络。
- 组件 0.1.3 起：`groupPolicy=disabled` 对 owner 同样生效；频道默认触发方式为 mention。
- 机器人在频道里的回复所有成员可见，隐私内容请走私聊。
- **频道登记**：`groupPolicy=allowlist` 时只有登记过的频道会响应，owner 本人不受限制（未登记频道里 @ 也会回复）。其他成员要在某个频道（含私有频道）里用机器人时：先把机器人邀请进频道，再由管理端登记：`node ~/zylos/.claude/skills/slack/src/admin.js add-group <频道ID> <名称>`。
- **smart / mention 切换**：管理端执行 `set-group-mode smart` 设置默认模式，或登记频道时在 add-group 末尾加模式单独设置某个频道；配置即时生效，无需重启。smart 模式下机器人会参考频道里之前的消息，有人提问会回，闲聊不插话（10-19 实测）。
- **文件收发**：App 需要 `files:read`、`files:write`（加完要 Reinstall to Workspace，Token 不变）。收文件目前依赖本地修复，**每次升级 Slack 组件后**给机器人发一张图片复测，收不到就重新打补丁或等研发修复版本（见踩坑第 8 条）。中文文件名会变成下划线，不影响内容。
