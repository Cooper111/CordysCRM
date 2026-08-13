---
type: domain-base
sub-type: api-index
module: follow
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/follow/controller 目录（或 crmfollow 包，核对代码）
owner: cordys-crm
verify-required: true
verify-note: 跟进计划类型（日常/事件/Todo）、needsRemind、提醒频率参数 易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/follow.md
  - knowledge/applications/backend/domain/base/domain-follow.md
---

# 跟进计划/记录 模块 API 索引（api-follow）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀（核对 permission.json） |
|---|---|---|---|
| **FollowUpRecordController**（通用） | 跟进记录通用（resourceType∈{CLUE,CUSTOMER,OPPORTUNITY,CONTRACT,ORDER}）：增改删查、图片/附件上传、某销售跟进统计 | FollowUpRecordService + AttachmentService | `{MODULE}:FOLLOW_RECORD_ADD/EDIT/DELETE/VIEW`（各业务模块独立权限） |
| **FollowUpPlanController**（通用） | 跟进计划通用（同上 resourceType）：增改删查、批量完成/批量延期、提醒生成 FollowReminderTask | FollowUpPlanService + FollowReminderTaskService | `{MODULE}:FOLLOW_PLAN_ADD/EDIT/DELETE/VIEW` |
| **FollowReminderTaskController** | 提醒任务：查询"我"的提醒列表、标记已读/批量已读、按 resourceType 过滤 | FollowReminderTaskService | `FOLLOW:REMIND_TASK_VIEW/MARK_READ` |

> 门面注意：clue/customer/opportunity/contract/order 各模块有自己的 `{Module}FollowPlanController / {Module}FollowRecordController`（例如 ClueFollowPlanController）——这些是"门面"，**内部代理到 Follow 通用 Service**。
> - 改跟进"通用逻辑" → FollowUpPlanService / FollowUpRecordService
> - 改某模块"特殊前置校验/返回结构" → 对应模块的 {Module}FollowXxxController

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| "跟进记录"列表/新建/编辑/附件上传 | 从某业务（线索/客户）Tab 进 → 对应模块 {Module}FollowRecordController；若全局 → FollowUpRecordController |
| "跟进计划"新建/日历/重复提醒 | 同上：{Module}FollowPlanController 或 FollowUpPlanController |
| 我的提醒 / 提醒中心 / 标记已读 | FollowReminderTaskController |

路径前缀（核对 @RequestMapping）：
```
/api/followup-record/*       → 通用跟进记录
/api/followup-plan/*         → 通用跟进计划
/api/follow-reminder-task/*  → 提醒任务
/api/{module}-follow-record/* → 各业务模块门面（如 clue-follow-record）
/api/{module}-follow-plan/*   → 各业务模块门面（如 opportunity-follow-plan）
```
