<div align="center">

# 🤖 AI 员工 · 应用连接知识库

**让 AI 员工（机器人）接入 IM、邮箱和各类业务应用的实战指南**

能不能接 · 怎么接 · 接上能做什么 · 为什么不行 —— 一份文档讲清楚

![apps](https://img.shields.io/badge/覆盖应用-28-blue)
![chapters](https://img.shields.io/badge/章节-30-green)
![updated](https://img.shields.io/badge/更新-2026--10--16-orange)
![lang](https://img.shields.io/badge/语言-中文-lightgrey)

</div>

---

## 📖 这是什么

这里收录了 AI 员工接入各类应用的**真实接入记录、踩坑记录和功能测试结果**，整理成适合人阅读、也适合 AI agent 学习检索的 Markdown 文档。

- **给人看**：部署人员照着步骤接入，客户成功同学查能力和限制
- **给 agent 学**：每个应用统一结构，开头附「给 Agent 的使用说明」，可直接作为知识库回答客户问题

> 文中所有密钥、Token、ID、内部地址均已隐去，截图中的敏感信息已打码。

## 🚀 快速开始

| 我想… | 去这里 |
|---|---|
| 通读全部内容 / 给 agent 当知识库 | [`app-connection-guide/README.md`](app-connection-guide/README.md)（单文件，完整版） |
| 只看某一个应用 | [`app-connection-guide/apps/`](app-connection-guide/apps/)（每章一个文件） |
| 先了解 owner、私聊 / 群聊权限等基础概念 | [通用概念](app-connection-guide/apps/02-common-concepts.md) |
| 接入出问题了 | [通用排障清单](app-connection-guide/apps/29-troubleshooting-checklist.md) |
| 接入前要找客户收集哪些信息 | [接入前信息收集表](app-connection-guide/apps/30-appendix-intake-form.md) |

## 🧭 应用一览

图例：✅ 已实测可用　⚠️ 部分可用 / 有已知限制　❌ 当前不可用　⬜ 未测试

### 💬 IM 聊天渠道

| 应用 | 状态 | 一句话说明 |
|---|:---:|---|
| [Microsoft Teams](app-connection-guide/apps/03-microsoft-teams.md) | ✅ | 私聊、群聊 @，需公网 HTTPS 回调 |
| [Slack](app-connection-guide/apps/04-slack.md) | ✅ | 私聊、频道 @，Socket Mode 无需公网 |
| [Discord](app-connection-guide/apps/05-discord.md) | ✅ | 私聊、服务器频道 @ |
| [Lark（国际版）](app-connection-guide/apps/06-lark.md) | ✅ | 私聊、群聊 @，可授权文档 / 表格 / 日历 |
| [飞书（国内）](app-connection-guide/apps/07-feishu.md) | ⚠️ | 私聊已验证；须用飞书企业账号 |
| [企业微信](app-connection-guide/apps/08-wecom.md) | ⚠️ | 私聊已验证；群里只收 @ 机器人的消息 |
| [钉钉](app-connection-guide/apps/09-dingtalk.md) | ⚠️ | 私聊已验证；群聊未测 |
| [WeChat（个人微信）](app-connection-guide/apps/10-wechat.md) | ✅ | 一对一私聊；不支持群聊 |
| [WhatsApp](app-connection-guide/apps/11-whatsapp.md) | ✅ | 扫码关联，私聊、群聊 @、引用回复 |
| [Telegram](app-connection-guide/apps/12-telegram.md) | ✅ | 私聊、群消息监控 |
| [Zalo](app-connection-guide/apps/13-zalo.md) | ✅ | 私聊、群聊 @；语音不能转文字 |
| [LINE](app-connection-guide/apps/17-line.md) | ✅ | 私聊、群聊 @、图片；推送计入月额度 |

### 📧 邮箱

| 应用 | 状态 | 一句话说明 |
|---|:---:|---|
| [Gmail](app-connection-guide/apps/14-gmail.md) | ✅ | 连接器或 IMAP/SMTP（推荐） |
| [QQ 邮箱](app-connection-guide/apps/15-qq-mail.md) | ✅ | 授权码 + IMAP/SMTP，收信监控、发信 |
| [Outlook](app-connection-guide/apps/18-outlook.md) | ✅ | 读信、收信、发信 |

### 🧩 应用连接器

| 应用 | 状态 | 一句话说明 |
|---|:---:|---|
| [GitHub](app-connection-guide/apps/16-github.md) | ✅ | 查看账号 / 仓库、拉取部署 |
| [Google Drive](app-connection-guide/apps/21-google-drive.md) | ✅ | 搜索、读写、整理、共享、评论 |
| [Google Search Console](app-connection-guide/apps/22-google-search-console.md) | ✅ | 搜索数据、站点地图；「检查网址」暂不可用 |
| [Google Analytics](app-connection-guide/apps/20-google-analytics.md) | ⚠️ | 可看账号 / 受众；暂不能查访问量报表 |
| [Instagram](app-connection-guide/apps/24-instagram.md) | ✅ | 帖子数据、发帖、读 / 回评论 |
| [Facebook](app-connection-guide/apps/23-facebook.md) | ❌ | 目前只能读取主页信息、帖子、照片 |
| [LinkedIn](app-connection-guide/apps/19-linkedin.md) | ⚠️ | 读资料、发帖、删帖；读帖子 / 评论暂不支持 |
| [Notion](app-connection-guide/apps/25-notion.md) | ✅ | 页面、数据库读写；评论暂不可用 |
| [Linear](app-connection-guide/apps/26-linear.md) | ✅ | 任务增删改查、标签、评论 |
| [Vercel](app-connection-guide/apps/27-vercel.md) | ✅ | 项目、部署、发布网页、环境变量 |
| [Zoho CRM](app-connection-guide/apps/28-zoho.md) | ✅ | 线索 / 联系人 / 客户 / 商机读写；不能删除 |

> 阿里邮箱未测；Amazon SP-API 暂无官方连接器。详见[概述](app-connection-guide/apps/01-overview.md)。

## 🗂️ 仓库结构

```text
.
├── README.md                     ← 你正在看的这页
└── app-connection-guide/
    ├── README.md                 ← 完整版（30 章，单文件）
    ├── apps/                     ← 每章一个文件，按编号排序
    │   ├── 01-overview.md
    │   ├── 02-common-concepts.md
    │   ├── …
    │   └── 30-appendix-intake-form.md
    └── images/                   ← 文中截图（已打码）
```

## 📌 使用须知

- **以实测为准**：标注「（官方流程，未实测）」的内容来自官方文档，使用前请以官方最新说明为准。
- **回答客户时**：先说能做什么，再说当前限制和替代办法；不承诺上线时间。
- **密钥安全**：Token、授权码等只通过私聊提供，切勿贴进群聊、文档或截图。
