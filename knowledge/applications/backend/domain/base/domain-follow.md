---
type: domain-base
sub-type: domain-index
module: follow
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/follow/ 目录（或 crmfollow 分包，核对代码）
owner: cordys-crm
verify-required: true
verify-note: 跟进计划类型/提醒方式/跟进状态 字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/follow.md
  - knowledge/applications/backend/domain/base/api-follow.md
---

# 跟进计划/记录 模块 领域模型索引（domain-follow）

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **FollowUpPlan** | 跟进计划主档：resourceType（CLUE/CUSTOMER/OPPORTUNITY/CONTRACT/ORDER）、resourceId、计划类型（日常/事件/TODO/周/月）、计划开始/结束时间、是否重复、重复 cron、提醒开关(needsRemind)、提醒提前量、负责人、计划状态（未开始/进行中/已完成/已取消/已逾期）、nextRemindTime | crm_follow_up_plan | FollowUpPlanExtMapper | N（跟进计划较少扩展） |
| **FollowUpRecord** | 跟进记录：resourceType、resourceId、跟进内容（富文本）、跟进方式枚举（电话/上门/微信/邮件/会议/其他）、跟进人、跟进时间、关联 计划 id（可选）、附件数量 | crm_follow_up_record | FollowUpRecordExtMapper | Y?（如允许自定义字段则有 RecordField+Blob，核对代码） |
| **FollowUpRecordAttachment** | 跟进记录附件：recordId、附件 URL、文件名、文件大小、附件类型（图片/文档/录音） | crm_follow_up_record_attachment（或走统一 Attachment 表，核对） | AttachmentMapper（统一） | N |
| **FollowReminderTask** | 提醒任务表（闹钟实现）：关联 planId、userId、remindAt（精确执行时间）、状态（待触发/已触发/已取消/失败）、触发方式（站内信/IM/短信/邮件 JSON） | crm_follow_reminder_task | FollowReminderTaskExtMapper | N |
| **FollowUpPlanRepeatInstance**（如有） | 重复计划的实例展开表（一个月计划每个月一行，便于查询某天计划）：planId、instanceDate、instanceStatus | crm_follow_plan_instance（核对） | FollowUpPlanInstanceExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.follow.dto.*` 或 `cn.cordys.crm.followup.dto.*`（核对包名）

---

## 二、定位技巧

| 关键词 | 类 |
|---|---|
| 跟进计划 / 下次跟进时间 / 重复计划 | FollowUpPlan |
| 今天写的跟进内容 / 谁跟进的 / 跟进方式 | FollowUpRecord |
| 跟进记录 附件 / 上传了哪些图片录音 | FollowUpRecordAttachment（或通用 Attachment 表+附件模块，核对） |
| "提醒我明天上午 10 点打电话" / 通知任务表 | FollowReminderTask |
| 重复计划 某一天 有没有展开 / 日历视图 | FollowUpPlanRepeatInstance（若独立） |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **ResourceTypeEnum** | 资源类型（CLUE/CUSTOMER/OPPORTUNITY/CONTRACT/ORDER 等，通用枚举所有模块都用） |
| **FollowUpPlanTypeEnum** | 计划类型（日常/事件/TODO/周计划/月计划） |
| **FollowUpPlanStatusEnum** | 计划状态（未开始/进行中/已完成/已取消/已逾期） |
| **FollowUpMethodEnum** | 跟进方式（电话/上门/微信/邮件/会议/其他，常走字典） |
| **RemindChannelEnum** | 提醒渠道（站内信/钉钉/飞书/企微/短信/邮件） |
