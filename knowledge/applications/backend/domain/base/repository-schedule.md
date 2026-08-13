---
type: domain-base
sub-type: repository-index
module: schedule
app: backend
status: official
evidence: backend/crm/src/main/resources/db/migration/*qrtz*.sql（若有） + backend/crm/src/main/java/cn/cordys/crm/schedule/ 目录
owner: cordys-crm
verify-required: true
verify-note: Quartz 表结构随版本变，业务调度 Job 列表 易变，核对代码 + application.yml quartz 配置
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/repository-core.md
  - knowledge/applications/backend/domain/base/repository-system.md
---

# 调度/定时任务 存储索引（repository-schedule）

> Quartz 本身有 11 张框架表，加上本项目的"业务级调度任务配置表" sys_schedule。本文件只写定位入口，Quartz 表字段不写。

---

## 一、Quartz 框架表（qrtz_*，11 张）

| 表名前缀 | 职责一句话 | 是否有 Mapper |
|---|---|---|
| qrtz_job_details | Job 详情（JobKey / JobClass / JobDataMap） | 无，走 Quartz JDBCJobStore（不直接 Mapper 操作） |
| qrtz_triggers | 触发器主表 | 同上 |
| qrtz_cron_triggers | Cron 触发器（cron 表达式） | 同上 |
| qrtz_simple_triggers | Simple 触发器（重复次数/间隔） | 同上 |
| qrtz_blob_triggers | BLOB 触发器（自定义 Trigger 序列化） | 同上 |
| qrtz_calendars | 日历（排除节假日等） | 同上 |
| qrtz_paused_trigger_grps | 暂停的触发器组 | 同上 |
| qrtz_fired_triggers | 已触发的触发器（运行中实例） | 同上 |
| qrtz_scheduler_state | Scheduler 状态（集群心跳） | 同上 |
| qrtz_locks | 行锁表（集群抢占锁） | 同上 |
| qrtz_simprop_triggers | SimplePropertyTrigger（日历触发等扩展） | 同上 |

> 操作 qrtz_* 表的正确方式：通过 Spring 的 `Scheduler` Bean API（scheduleJob / rescheduleJob / pauseJob / deleteJob），**禁止直接用 Mapper 写 qrtz_* 表**（会破坏 Quartz 状态一致性 R3-B）。

### Flyway 脚本位置
```
db/migration/V{ver}__init_qrtz_tables.sql（Quartz 官方 DDL 脚本，版本对应 Quartz 2.x / 3.x）
```
**严禁修改 qrtz_* 建表脚本**（一旦生产跑过，改脚本会 Flyway checksum 失败 R5-B）。

---

## 二、业务级调度任务配置表（sys_schedule）

| ExtMapper 类名 | 职责一句话 | 表名（核对 @TableName） |
|---|---|---|
| **ExtScheduleMapper** | 业务层自定义调度任务配置：jobKey、cron、jobClassName、status(启用/停用)、lastExecuteTime、lastExecuteResult、remark | sys_schedule（核对） |

> 说明：如果项目用的是"配置中心 + @Scheduled"模式而不是 sys_schedule 表，则此表可能不存在——核对 crm 包下是否有 schedule 模块即可。

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| "这个任务的 Cron 在哪里配" | ① 查 sys_schedule 表 ExtScheduleMapper；② application.yml 的 cordys.cron.* 配置；③ 类上的 @Scheduled(cron=…) 注解（三种方案用了哪种，核对代码） |
| 线索自动回收 Job 实现类 | 找 Clue 模块的 `*Job.java` / `*RecycleListener.java`（如 CluePoolRecycleJob） |
| 提醒任务扫描 Job（每天 08:00 生成今日提醒） | FollowReminderScanJob（follow 模块） |
| 数据范围权限 / 操作日志 刷新缓存的定时任务 | 找 framework.system 的 `*CacheRefreshJob` 类 |
