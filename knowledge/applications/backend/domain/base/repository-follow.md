---
type: domain-base
sub-type: repository-index
module: follow
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/follow/mapper/
owner: cordys-crm
verify-required: true
verify-note: 跟进记录附件 可能走统一 Attachment 表，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-follow.md
---

# 跟进模块 存储索引（repository-follow）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀（核对 @TableName） |
|---|---|---|
| **ExtFollowUpPlanMapper** | 跟进计划 CRUD/日历视图查询/我的提醒 | crm_follow_up_plan（或 follow_up_plan，核对） |
| **ExtFollowUpRecordMapper** | 跟进记录 CRUD/按 resourceType+resourceId 列表 | crm_follow_up_record |
| **ExtFollowUpRecordAttachmentMapper**（若独立，或走统一 AttachmentMapper） | 跟进记录附件列表 | crm_follow_up_record_attachment 或 sys_attachment（核对代码） |
| **ExtFollowReminderTaskMapper** | 提醒任务表（闹钟） | crm_follow_reminder_task |
| **ExtFollowUpPlanInstanceMapper**（若独立） | 重复计划展开的实例表 | crm_follow_plan_instance |

XML 路径：`resources/mapper/follow/Ext{Xxx}Mapper.xml`

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_follow_tables.sql`

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 某客户的跟进记录 按时间倒序 | ExtFollowUpRecordMapper.xml selectByResource |
| 明天要跟进的所有计划（日历 API） | ExtFollowUpPlanMapper.xml selectPlanBetweenDate，含重复计划展开（若有 instance 表则 JOIN instance） |
| 定时扫描生成提醒任务 / 扫描逾期未跟进 | follow 相关 Job + ExtFollowReminderTaskMapper.batchInsert |
| 跟进记录的图片/附件下载地址 | 看 attachment 表实现：FollowAttachment 独立表 或 通用 Attachment 模块（核对代码） |
