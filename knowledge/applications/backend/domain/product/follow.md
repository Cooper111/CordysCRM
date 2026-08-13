---
type: domain-product
app: backend
module: follow
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/follow 下：FollowUpPlan + FollowUpRecord + Comment（各自的 Service/Controller/Mapper/Field/Blob），以及 BaseFollowUpService 基础抽象（clue/customer/opportunity 复用）
owner: cordys-crm
verify-required: true
verify-note: 跟进类型、提醒策略、评论@通知、计划状态需回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#跟进计划/跟进记录/评论
  - knowledge/main/state-defs.md#三跟进计划状态
  - knowledge/main/business-rules.md#6-通知触发
  - knowledge/applications/backend/domain/base/api-follow.md
  - knowledge/applications/backend/domain/base/domain-follow.md
  - knowledge/applications/backend/domain/base/repository-follow.md
---

# 跟进模块 · 产品能力

---

## 一、能力概述

跟进模块提供"销售动作留痕"的通用能力，将业务对象（线索/客户/商机）的跟进分为**跟进计划（下一步什么时候做什么）**和**跟进记录（实际做了什么）**两类实体，均支持自定义字段 + 评论 + @通知 + 提醒。

**设计模式（关键）**：跟进是「跨资源通用能力」，通过 resourceType + resourceId 两字段与 3 个业务模块耦合：
- resourceType = CLUE / CUSTOMER / OPPORTUNITY
- resourceId = 对应线索/客户/商机的主键 ID

对应 Controller：线索/客户/商机模块内各自提供 `{Xxx}FollowPlanController` + `{Xxx}FollowRecordController`，内部代理 follow.*Service（同一套 Service）。

主干流程：
```
创建跟进计划（绑定资源+时间+提醒）→ 到点提醒（Quartz监听器发通知）
         ↓ 到点实际跟进
   创建跟进记录（关联对应计划/或独立）+ 评论（0..N 条）
         ↓
   计划状态流转：待处理→已完成/已逾期/已取消
```

---

## 二、核心流程（ASCII 流程图）

```
                  ┌──── resourceType = CLUE / CUSTOMER / OPPORTUNITY ────┐
                  │                                                     │
   ┌──────────────┴─────────────── Plan 跟进计划 ────────────────────────┴──────────────┐
   │                                                                                     │
   │  创建 Plan (FollowUpPlanService.createFor{Clue/Customer/Opportunity})               │
   │   ├ planType：FollowUpPlanType（电话/拜访/会议/... 字典）                             │
   │   ├ remindAt / remindBefore（提前多少分钟提醒）                                       │
   │   └ 计划状态=PENDING（待处理）                                                       │
   │                         │                                                            │
   │                         ▼ 提醒调度                                                     │
   │           FollowUpPlanRemindListener（Quartz）                                        │
   │               if (NOW >= plan.remindAt)                                              │
   │               then → 站内信 + 邮件 + 企微/钉钉/飞书（按 Notice 配置）→ 通知 owner       │
   │                         │                                                            │
   │           ┌─────────────┴────────────┬──────────────┐                                │
   │           ▼                          ▼              ▼                                │
   │      已完成（主动点完成）         已逾期(NOW>endAt)  已取消（计划失效）                 │
   │   status=COMPLETED             status=OVERDUE    status=CANCELLED                    │
   │           │                                                                          │
   └───────────┴──────────────────────────────────────────────────────────────────────────┘
                         │ 销售点"写跟进记录"
                         ▼
   ┌─────────────── Record 跟进记录 ─────────────────┐
   │  FollowUpRecordService.create                   │
   │   ├ 可选：关联 planId（记录由计划产生）            │
   │   ├ 内容 + 附件 + 自定义字段                      │
   │   └ 评论 Comment（0..N 条，支持 @提醒）            │
   │         └ CommentService → @某人 → 通知          │
   └──────────────────────────────────────────────────┘
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 跟进计划创建 | {线索/客户/商机}详情 Tab→计划→新建 | 写 Plan + PlanField + PlanFieldBlob → owner=当前用户 → 写入调度提醒（remindAt） → 操作日志 | FollowUpPlanService.add（BaseFollowUpService 封装通用逻辑，按 resourceType 区分） |
| 跟进计划提醒 | Quartz 定时触发 FollowUpPlanRemindListener | 扫描 remindAt <= NOW + 未通知过的计划 → 负责人/协作人通知（NoticeService）→ 标记已通知（避免重复） | FollowUpPlanRemindListener + CommonNoticeSendService |
| 跟进计划状态流转 | 点完成 / 点取消 / 到点逾期监听器 | 写 Plan.status（FollowUpPlanStatusType 枚举）→ 逾期自动触发监听器（每日扫描一次） | FollowUpPlanService.updateStatus + FollowUpPlanStatusType 状态机 |
| 跟进记录创建 | 计划 Tab→写跟进 / 详情 Tab→新增记录 | 写 Record + RecordField + RecordFieldBlob → 可选关联 planId → 自动标记计划=已完成（若关联）→ 通知相关人 | FollowUpRecordService.add（BaseFollowUpService） |
| 评论 / @某人 | 计划/记录详情→评论 Tab | 写 Comment + 如果评论内容含 @用户 → 解析被@人 → 发被@通知（站内信+邮件+IM）→ 评论可编辑/删除（本人） | BaseCommentService + FollowUpPlanCommentService + FollowUpRecordCommentService |
| 首页待跟进统计 | 首页"我的待跟进"卡片 | HomeStatisticService → 按当前用户 owner 查 Plan：今日 + 未完成 + 已逾期分类汇总（Today/Week/Month） | HomeStatisticService.countMyPlans（按 HomeStatisticPeriod） |

---

## 三、核心业务规则

| 规则 ID | 规则描述（if/then） | 例外 |
|---|---|---|
| R1 | **归属权限**：if (当前用户不是 plan/record.owner AND 不是协作人 AND 不是管理员) then 禁止 UPDATE/DELETE；读权限受 resourceType 级别的资源 DataScope 控制 | 管理员可编辑/删除任意 |
| R2 | **逾期自动判定**：if (NOW > plan.endAt AND plan.status = PENDING) then 监听器标 OVERDUE（不做物理处理，只改状态） | 组织参数开关："跟进计划自动逾期"=关则停止此逻辑 |
| R3 | **@提醒人数上限**：单条评论 if (@人数 > 20) then 拒绝发送，提示"@太多人，请分批发通知" | 管理员不限 |
| R4 | **记录附件大小限制**：沿用 system 通用文件大小上限（默认 50MB/单个）；单条记录附件 <= 20 个 | 超管配置 Parameter.FOLLOW_UP_ATTACH_LIMIT |
| R5 | **删计划不删关联记录**：if (删除 Plan) then 关联的 Record 不级联删除，改为 planId = null（独立记录展示） | 勾选"同时删除记录"才会级联（需二次确认） |
| R6 | **评论防刷屏**：if (某用户 1 分钟内评论 > 20 条 on 同一资源) then 限流 10s 再发；同时操作日志标记异常 | 管理员不限 |
| R7 | **跨资源协作可见性**：A 是客户协作人 → 可看该客户下全部 Plan + Record（不区分是否自己是 owner）；对应 BaseFollowUpService.getDataScope 过滤条件 | — |

---

## 四、状态/枚举（定位入口）

| 语义 | 枚举/类名入口 | 代码路径 |
|---|---|---|
| 跟进计划状态（待处理/已完成/已逾期/已取消） | `cn.cordys.crm.follow.constants.FollowUpPlanStatusType` | crm.follow.constants.FollowUpPlanStatusType |
| 跟进类型（电话/拜访/会议/邮件/其他） | `FollowUpPlanType` 枚举 + 字典 DictModule.FOLLOW_UP_PLAN_TYPE | crm.follow.constants.FollowUpPlanType + DictModule |
| 资源类型（本模块绑定哪类业务对象） | `ModuleKey.CLUE / CUSTOMER / OPPORTUNITY`（common.constants.ModuleKey） | common.constants.ModuleKey |
| 表单 Key | `FormKey.FOLLOW_UP_PLAN / FOLLOW_UP_RECORD / COMMENT` | common.constants.FormKey |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **clue / customer / opportunity** | follow ← 3 模块（Controller 层代理） | {Clue/Customer/Opportunity}FollowPlanController 代理 FollowUpPlanService，传 resourceType + resourceId；字段、权限、容量、协作都依赖上游模块 |
| **system（通知 Notice）** | follow → system | 计划到点提醒 / @人通知 → CommonNoticeSendService（站内信+邮件+IM）；通知事件 = NotificationConstants.FOLLOW_UP_* |
| **system（首页 HomeStatistic）** | follow → system | HomeStatisticService 调 ExtFollowUpPlanMapper.countXxx 得到首页待跟进/逾期统计卡片 |
| **system（模块字段）** | follow → system | FollowUpPlanField / RecordField 由 system.ModuleFieldService 管理（BusinessModuleField.PLAN / RECORD） |
| **system（操作日志）** | follow → system | FollowUpPlanLogService / FollowUpRecordLogService（extends BaseModuleLogService）→ 操作日志前后 diff |
| **system（调度 Quartz）** | follow → system | FollowUpPlanRemindListener（system.job.listener）注册到 Quartz；NoticeExpireJob 清理过期通知 |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **提醒重复发送（服务重启/抖动）** | 每次发完提醒 → 在 plan 上写 remindSent=true + remindSentAt；监听器发送前强校验：只处理 remindSent=false 的 | FollowUpPlanRemindListener before-send 条件 |
| **跟进记录内容有敏感词** | 统一接入 system/ContentCensor（如未来有）；目前无内建敏感词过滤 → 操作日志+审计留痕 | （预留扩展点） |
| **@用户离职（已停用）** | 解析@时先校验 User.status=启用；停用用户跳过发送 → 在评论下方标"已跳过 N 个离职用户" | BaseCommentService.resolveAtUsers 过滤停用 |
| **导出跟进记录（按客户）** | 客户详情→更多→导出跟进：使用 FastExcel 分页查询 + 流式写（小量同步大量异步 ExportTask） | CustomerController → CustomerExportService.exportFollowUps（或类似入口） |
| **同一资源计划数过多（500+）** | Plan 查询列表走分页；默认只查最近 30 天；"查看全部"提供按创建时间倒序+分页 | FollowUpPlanService.page + PageHelper |
| **协作人看不到客户的跟进**（新协作者刚加入） | 协作人加入前的跟进记录：默认仍可见（"协作人可见全部历史跟进"策略开关在 OrganizationConfig 可配置）；可配置为"加入后的记录才可见" | BaseFollowUpService.getDataScope + OrganizationConfigService |
