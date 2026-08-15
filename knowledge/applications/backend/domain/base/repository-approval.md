---
type: domain-base
sub-type: repository-index
module: approval
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/approval/mapper/
owner: cordys-crm
verify-required: true
verify-note: 审批节点 JSON/流变量/候选人逻辑 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-approval.md
---

# 审批模块 存储索引（repository-approval）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀（实际名核对 @TableName） |
|---|---|---|
| **ExtApprovalFlowMapper** | 审批定义 CRUD | crm_approval_flow（或 approval_flow，核对） |
| **ExtApprovalFlowVersionMapper**（若独立） | 审批定义多版本 | crm_approval_flow_version |
| **ExtApprovalInstanceMapper** | 审批实例 CRUD/分页 | crm_approval_instance |
| **ExtApprovalInstanceNodeMapper** | 实例节点轨迹 | crm_approval_instance_node |
| **ExtApprovalInstanceTaskMapper** | 待办任务 CRUD/我的待办分页 | crm_approval_instance_task |
| **ExtApprovalActionLogMapper** | 操作流水日志（同意/拒绝/加签…） | crm_approval_action_log |
| **ExtApprovalNodeCandidateMapper** | 节点候选人配置 | crm_approval_node_candidate |
| **ExtApprovalInstanceCcMapper** | 抄送人 | crm_approval_instance_cc |

XML 路径：`resources/mapper/approval/Ext{Xxx}Mapper.xml`

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_approval_tables.sql`

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 我的待办 SQL（跨业务类型，按当前 userId + 待办状态） | ExtApprovalInstanceTaskMapper.xml selectMyTodoPage |
| 某审批 审批轨迹（每个节点谁做了什么） | ExtApprovalActionLogMapper.xml selectByInstanceId |
| 为什么 A 是这个节点的审批人 | ApprovalNodeCandidateMapper 配置表 + 运行时 Task 动态计算 SQL |
| 撤回 时 需要回滚哪些表 | 看 ApprovalService.withdraw() 的事务内 mapper 调用，核对 instance / task / node / action_log 四张表的状态变更 |
