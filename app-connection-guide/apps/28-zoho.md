# Zoho

> 摘自《应用连接文档（AI 员工接入各应用指南）》第 28 章（完整版见 [../README.md](../README.md)）。状态图例：✅ 已实测可用；⚠️ 部分可用 / 有已知问题；❌ 当前不可用；⬜ 未测试。

Zoho 是海外常用的一整套企业 SaaS 软件（CRM、财务记账、开票、客服工单、库存、企业邮箱等），其中最常用的是 **Zoho CRM**（客户关系管理），用来管销售线索、联系人、客户公司和商机，类似国内的「纷享销客、销售易」或海外的 Salesforce、HubSpot。CRM 基本概念：**线索（Leads，还没确认的潜在客户）**、**联系人（Contacts）**、**客户（Accounts，即客户公司）**、**商机（Deals，一笔正在跟进的生意，带阶段）**；线索确认后可「转化」成联系人 + 客户。OpenMax 里 Zoho 共有 7 个连接器，本章介绍的是 **OpenMax Zoho 连接器（即 Zoho CRM）**，OAuth 授权，**代理模式（proxy）**，共 6 个动作。实测：**读写全部通过（6/6），可用于线索录入 / 更新、线索转化、商机与联系人关联；连接器不能删除记录**。

## 能做什么

- OpenMax 的 7 个 Zoho 连接器：Zoho CRM `zoho`（6 个动作，本章所测）、Zoho Bigin `zoho_bigin`（面向小团队的简版 CRM，12 个动作，未测）、Zoho Invoice `zoho_invoice`（开票，6 个动作，只读，未测）；Zoho Books / Desk / Inventory / Mail（`zoho_books` / `zoho_desk` / `zoho_inventory` / `zoho_mail`）4 个连接器目前 **暂无可用功能（0 个动作）**，财务、客服、库存、邮箱场景请在对应的 Zoho 网页后台操作。
- 读：按模块读取记录 `ZOHO_GET_ZOHO_RECORDS`，模块可选线索 Leads / 联系人 Contacts / 客户 Accounts / 商机 Deals，可指定只返回部分字段。
- 写：新建记录 `ZOHO_CREATE_ZOHO_RECORD`（线索、商机等）、修改记录 `ZOHO_UPDATE_ZOHO_RECORD`、新建标签 `ZOHO_CREATE_ZOHO_TAG`。
- 线索转化：把线索转成联系人 + 客户 `ZOHO_CONVERT_ZOHO_LEAD`。
- 关联：更新相关列表 `ZOHO_UPDATE_RELATED_RECORDS`，如把联系人加到商机的「联系人角色」里并指定角色（如决策人）。
- ❌ 不能删除记录：连接器没有删除动作，删除记录 / 标签只能在 Zoho 网页上手动操作。
- ⚠️ 线索转化不会同时建商机：转化只生成联系人 + 客户，需要商机时单独新建并关联（见踩坑）。

## 前置条件 / 所需权限 / 账号

- 一个 **Zoho 国际版账号**（zoho.com，邮箱免费注册，开通 Zoho CRM 免费版 / 试用版即可）。**不要用中国站 zoho.com.cn 的账号**：两边账号和数据不互通，连接器对接的是国际版（区域 com）。
- 新开通的 CRM 自带一批示例数据（名字带「(Sample)」），可直接用来试读。
- 授权账号在 CRM 里能做的事，agent 就能做（权限跟随账号）。

## 详细接入步骤（已验证）

1. 在 zoho.com 注册 / 登录国际版账号，开通 Zoho CRM。
2. OpenMax 工作台 →「应用连接」→ Zoho（即 Zoho CRM）→ 登录 Zoho 国际版账号 → 同意授权（区域为 com）。
3. 连接成功后（proxy 模式）agent 即可使用。
4. 机器人调 `ZOHO_GET_ZOHO_RECORDS` 读线索（Leads）验证：能读到示例线索即连接成功。

## 测试清单

| 测试项 | 方法 | 实测结果 |
|---|---|---|
| 读线索 | GET_ZOHO_RECORDS（module_api_name=Leads，fields 可选） | ✅ 读到示例线索 |
| 读联系人 | GET_ZOHO_RECORDS（module_api_name=Contacts） | ✅ 读到示例联系人，带所属公司 |
| 读客户 | GET_ZOHO_RECORDS（module_api_name=Accounts） | ✅ 读到示例客户 |
| 读商机 | GET_ZOHO_RECORDS（module_api_name=Deals） | ✅ 读到示例商机，带阶段（如「识别决策人」） |
| 新建线索 | CREATE_ZOHO_RECORD（module_api_name + data[]） | ✅ |
| 修改线索（公司名 / 电话） | UPDATE_ZOHO_RECORD（data[{id, ...}]） | ✅ 回读确认已生效 |
| 新建标签 | CREATE_ZOHO_TAG（module_api_name + name） | ✅ |
| 线索转化 | CONVERT_ZOHO_LEAD（lead_id） | ✅ 生成联系人 + 客户，原线索随即消失；⚠️ 不会自动建商机（见踩坑） |
| 新建商机（关联客户） | CREATE_ZOHO_RECORD（module_api_name=Deals，Account_Name:{id}） | ✅ |
| 把联系人加到商机的联系人角色 | UPDATE_RELATED_RECORDS（module_api_name、record_id、related_list_api_name=Contact_Roles、data[{id, Contact_Role:{name}}]） | ✅ 联系人以「决策人」角色关联到商机 |
| 删除记录 | — | ❌ 连接器无删除动作，须在 Zoho 网页手动删除 |

## 踩坑记录与解决办法

- **参数名是 module_api_name，不是 module**：读、建、改、建标签都要传 `module_api_name`（Leads / Contacts / Accounts / Deals），写成 module 调用不成功。
- **线索转化不会自动建商机**：`ZOHO_CONVERT_ZOHO_LEAD` 虽然有 deal_name / deal_stage 参数，但实测不生效，只生成联系人 + 客户，商机为空（Deals=null）。需要商机时，转化后用 `ZOHO_CREATE_ZOHO_RECORD` 单独建（module_api_name=Deals，Account_Name 填转化出的客户），再用 `ZOHO_UPDATE_RELATED_RECORDS` 把联系人加到商机的联系人角色里。
- **没有删除功能**：连接器 6 个动作里没有删除，误建或不再需要的记录只能在 Zoho 网页手动删：联系人、客户、商机在各自模块里删；标签在「设置 → 标签」里删。
- **Zoho Books / Desk / Inventory / Mail 连接器 0 个动作**：能连接，但连接后没有任何可用功能，财务、客服、库存、邮箱场景暂不支持。替代办法：在对应的 Zoho 网页后台操作。
- **账号要用国际版 zoho.com**：中国站 zoho.com.cn 的账号不要用，授权区域为 com。
- **线索转化后原线索消失**：转化后在线索（Leads）里查不到原线索（已变成联系人 + 客户），属 Zoho 正常行为。
- **agent 侧注意**：修改记录时 data 是数组，每条都要带记录 id；新建商机关联客户用 Account_Name:{id}；加联系人角色时 record_id 填商机、data 里的 id 填联系人。

## 日常运维 / 注意事项

- 授权过期或 Zoho 账号改密码后，需在 OpenMax 工作台重新授权。
- 建议口径：查线索 / 联系人 / 客户 / 商机可直接做；新建、修改、线索转化先说明再做；连接器删不了数据，误建的记录要人工到 Zoho 网页删除。
- 客户问「连上 Zoho 能干嘛」：把新客户 / 展会名片直接录成 CRM 线索、更新跟进信息、线索确认后转成联系人 + 客户、建商机并挂上决策人，随时查线索 / 客户 / 商机清单。
