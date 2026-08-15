---
type: domain-base
sub-type: domain-index
module: approval
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/approval/ 目录
owner: cordys-crm
verify-required: true
verify-note: 审批节点类型/流变量 JSON/加签类型 字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/approval.md
  - knowledge/applications/backend/domain/base/api-approval.md
---

# 审批模块 领域模型索引（domain-approval）

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **ApprovalFlow** | 审批定义（流程模板）：flowKey、flowName、关联业务类型 form_key（如 CUSTOMER_TRANSFER / CONTRACT_SIGN / BUSINESS_TITLE）、流程 JSON（设计器节点结构）、发布版本号、状态（草稿/已发布/已停用）、创建人 | crm_approval_flow | ApprovalFlowExtMapper | N |
| **ApprovalFlowVersion**（若独立） | 审批定义多版本：flowId、version、流程 JSON 快照、是否当前版本、发布时间、发布人 | crm_approval_flow_version（核对） | ApprovalFlowVersionExtMapper | N |
| **ApprovalInstance** | 审批实例：instanceId、flowId、flowVersion、业务类型(bizType)、业务主键(bizId)、发起人、发起时间、当前状态（审批中/已通过/已拒绝/已撤回/已作废）、当前节点号 JSON | crm_approval_instance | ApprovalInstanceExtMapper | N |
| **ApprovalInstanceNode** | 审批实例的 节点 实例：instanceId、nodeId、nodeType（开始/结束/用户任务/条件分支/并行分支/会签/或签）、节点状态（未到达/待处理/已通过/已拒绝/已跳过）、到达时间、处理时间 | crm_approval_instance_node | ApprovalInstanceNodeExtMapper | N |
| **ApprovalInstanceTask** | 审批待办任务：关联 instanceId + nodeId + taskId、待办人 userId、任务状态（待办/已办/已转办/已加签/已会签未完成…）、任务创建时间、完成时间、操作意见（同意/拒绝/加签/转办/退回） | crm_approval_instance_task | ApprovalInstanceTaskExtMapper | N |
| **ApprovalActionLog** | 审批操作流水日志：instanceId、taskId（可选）、操作人、action 类型（SUBMIT/AGREE/REJECT/DELEGATE/ADD_SIGN/COUNTER_SIGN/WITHDRAW/RETURN）、意见内容、操作时间、操作前后节点快照 JSON | crm_approval_action_log | ApprovalActionLogExtMapper | N |
| **ApprovalNodeCandidateUser** | 节点候选人配置（审批定义级）：flowId + nodeId、候选来源类型（指定用户/指定角色/部门负责人/上一节点操作人/发起人主管…）、候选值 JSON（用户 id 列表 / 角色 id / 主管层级数字） | crm_approval_node_candidate（核对表名） | ApprovalNodeCandidateExtMapper | N |
| **ApprovalInstanceCc** | 审批抄送人：instanceId、ccUserId、是否已读、抄送人配置来源节点 id | crm_approval_instance_cc | ApprovalInstanceCcExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.approval.dto.*`

---

## 二、定位技巧

| 关键词 | 类 |
|---|---|
| 审批流程模板 / 流程设计 / 发布 / 启用停用 | ApprovalFlow + ApprovalFlowVersion（若独立） |
| 某合同的 审批流程 现在走到哪了 / 审批中 / 已通过 | ApprovalInstance |
| 某审批流 每个节点 什么状态 / 哪个节点卡住了 | ApprovalInstanceNode |
| 谁有我的待办 / 待办列表 / 我审批过什么 | ApprovalInstanceTask |
| 谁在什么时候 点了同意 / 加签 / 撤回 / 意见是什么 | ApprovalActionLog |
| 某节点 谁可以审批 / 为什么 A 能看到 B 看不到 | ApprovalNodeCandidateUser（定义级）+ 运行时计算的 Task 表 |
| 审批抄送 / 抄送我了 | ApprovalInstanceCc |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **ApprovalStatusEnum** | 审批实例状态（审批中/已通过/已拒绝/已撤回/已作废） |
| **ApprovalNodeTypeEnum** | 节点类型（开始/结束/用户任务/条件/并行/会签/或签…） |
| **ApprovalNodeStatusEnum** | 节点状态 |
| **ApprovalTaskStatusEnum** | 待办任务状态 |
| **ApprovalActionTypeEnum** | 操作类型（AGREE/REJECT/DELEGATE/ADD_SIGN/COUNTER_SIGN/WITHDRAW/RETURN…） |
| **CandidateSourceTypeEnum** | 候选人来源（指定人/角色/部门主管/发起人上级…） |
| **ApprovalBizTypeEnum** | 审批业务类型枚举（CONTRACT_SIGN / BUSINESS_TITLE / CUSTOMER_MERGE / CLUE_ASSIGN…，对应 form_key，核对代码） |
