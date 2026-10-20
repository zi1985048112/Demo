# Instagram

> 摘自《应用连接文档（AI 员工接入各应用指南）》第 24 章（完整版见 [../README.md](../README.md)）。状态图例：✅ 已实测可用；⚠️ 部分可用 / 有已知问题；❌ 当前不可用；⬜ 未测试。

Instagram（简称 IG）是 Meta 旗下的图片 / 短视频社交平台，可以理解为海外版的「小红书 + 抖音」：发图文帖、短视频（Reels），和粉丝通过评论、私信互动，品牌和商家常用来做海外营销。通过 **OpenMax Instagram 连接器**接入，OAuth 授权，**代理模式（proxy）**，共 16 个动作。实测：**读数据、发帖、评论全部通过**；私信暂不测试。

## 能做什么

- 发帖：建草稿（单图 / 视频 / Reels）`INSTAGRAM_CREATE_MEDIA_CONTAINER`、轮播草稿 `CREATE_CAROUSEL_CONTAINER`、查草稿处理状态 `GET_POST_STATUS`、发布 `CREATE_POST`。
- 看数据：账号资料 `GET_USER_INFO`（粉丝 / 关注 / 帖子数）、帖子列表 `GET_USER_MEDIA`、单帖数据 `GET_POST_INSIGHTS`、账号数据 `GET_USER_INSIGHTS`。
- 评论：评论列表 `GET_POST_COMMENTS`、回复评论 `REPLY_TO_COMMENT`。
- 私信：会话列表 / 会话详情 / 消息列表 `LIST_ALL_CONVERSATIONS` / `GET_CONVERSATION` / `LIST_ALL_MESSAGES`、发文字 / 图片私信 `SEND_TEXT_MESSAGE` / `SEND_IMAGE`、标记已读 `MARK_SEEN`。
- ❌ 不能删帖、改帖（连接器没有这两个动作）。替代办法：在 Instagram App 里手动删除或编辑。另：OpenMax 新上线官方 Meta（Facebook / Instagram）连接器，暂不测试（10-19）。

## 前置条件 / 所需权限 / 账号

- 只支持**专业账号**：商业号（Business）或创作者号（Creator）。**个人号连不上**。
- 个人号切换方法：Instagram App → 设置 → 账号类型和工具 → 切换为专业账号（免费，可随时切回）。（实测使用创作者号。）
- 本连接器走 Instagram 账号登录授权，实测不需要绑定 Facebook 公共主页。

## 详细接入步骤（已验证）

1. （如是个人号）先在 Instagram App 里切换为专业账号。
2. OpenMax 工作台 →「应用连接」→ Instagram → 登录 Instagram 账号并同意授权。
3. 授权给 AI 员工，机器人收到「连接已授权」通知（proxy 模式），连接成功后 agent 即可使用。
4. 机器人调 `GET_USER_INFO` 验证：能返回账号资料（粉丝 / 关注 / 帖子数）即连接成功。

## 测试清单

| 测试项 | 方法 | 实测结果 |
|---|---|---|
| 账号资料（粉丝 / 关注 / 帖子数） | GET_USER_INFO | ✅ |
| 帖子列表 | GET_USER_MEDIA | ✅ |
| 账号数据 | GET_USER_INSIGHTS | ✅ 触达（reach，按天）；⚠️ profile_views、accounts_engaged 等汇总类（total_value）指标返回空（可能是新号暂无数据或需另传参数） |
| 单帖数据（触达 / 点赞 / 评论 / 收藏 / 分享） | GET_POST_INSIGHTS | ✅ |
| 发帖 | CREATE_MEDIA_CONTAINER → GET_POST_STATUS（FINISHED）→ CREATE_POST | ✅ |
| 读评论（含评论人 / 时间） | GET_POST_COMMENTS | ✅ |
| 回复评论 | REPLY_TO_COMMENT | ✅ 回复显示在原评论下 |
| 私信会话列表 | LIST_ALL_CONVERSATIONS | ✅ 接口正常（列表为空） |
| 私信读 / 回 / 标已读 | LIST_ALL_MESSAGES / SEND_TEXT_MESSAGE / MARK_SEEN | 暂不测试（需另一个账号先发私信才能测） |

## 踩坑记录与解决办法

- **接口不能删帖**：连接器没有删帖 / 改帖动作，帖子须在 Instagram App 里手动删除。
- **发帖图片必须是公网可访问的图片链接（JPEG）**：不能直接传本地文件；建草稿后先查草稿状态，为 FINISHED 再发布。
- **私信受 Instagram 24 小时规则限制**：商家只能回复 24 小时内主动发来消息的用户，不能主动给陌生人群发。
- **个人号连不上**：须先切换为专业账号（商业号或创作者号）再授权。
- **agent 侧注意**：读评论接口返回的列表里，agent 自己的回复也会出现在顶层，统计评论数或判断「待回复」时要区分。

## 日常运维 / 注意事项

- 授权过期或 Instagram 改密码后，需在 OpenMax 工作台重新授权。
- 发帖、回复评论、发私信都是公开或对外可见的操作：正式发布前先让客户确认文案和图片，测试内容测完即删（删帖须在 App 里手动操作）。
- 客户问「连上 Instagram 能干嘛」：看粉丝和帖子数据、按确认好的文案和图片发帖、查看并回复评论、回复 24 小时内的粉丝私信。
