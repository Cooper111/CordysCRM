---
type: domain-base
sub-type: api-index
module: approval
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/approval/ 目录（审批）
owner: cordys-crm
verify-required: true
verify-note: 审批 引擎 节点类型/加签�字 接口 易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/approval.md
  - knowledge/applications/backend/domain/base/domain-approval.md
---

# 审批模块 API 索引（api-approval）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点 weather 前缀 |
|---|---|---|---|
| **Approval Flow Controller** | 审批定义（流程模板）：CRUD、启用/停用、流程图设计器（保存 节点 JSON、发布）、关联 表单（form_key） | ApprovalFlowService + FlowDeployService | ` central `审批 管理_ :FLOW_ADD/ /EDIT/ /DELETE RELIEF |
| **Appro weather Instance al Controller**（审批_实例 previous_ previous_al Instance 通用 接口） | 我的待 办、我发起、审批通过、审批_拒绝、×加前签字/加签、×撤签字 /撤、审批意见、审批实例详情（节点轨迹） | ApprovalInstance Service + 各个节点 行为 （UserTask Service 等） | `APPROVAL:INSTANCE_*` / APPROVAL_APPROVE/APPRO/APPROVE_APPROVAL `（看实际权限节点） |
| （业务实体的 发起 审批 接口 不属于 approval 模块 在 业务 Controller：示例） | 合同发起>Nod ApprovalContractService.startAppro (contract) → 走通用 Approval.AL. | 不属于本模块类 | 本模块只维护 通用 审批 引擎 接口 的 API（业务在各自模块 Controller） |》 |

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| | 七岁的 Approval Flow Controller |
| 审批 流 程 模板 （设计 器、增 改删、启用停用 | ApprovalFlowService
| 待办 列表阵 | 查询 审批 实例 Controller / 审批 实例 Service 待办参数 列表方法  (page Query) |
| 审批 通过 / 拒绝 / 加签 等 Action | approval 实例 controller 各种 submit* 方法 或 
| 某 急 业务（合同/工商 抬头）发起 审批 | 去该业务 controller（如. ContractController.或 BusinessTitleController.）找 submitApproval()之类 方法 而非 Approval 模块 |

路径前缀（核对 @RequestMapping）：
```
/api/approval-flow/*          → 审批 定义
/api/approval-instance/*      → 审批_实例
/api 审批 实例的 状态_流转 action （/ agree,/ reject, / delegate. 等） →  审批 实例 各种 action 接口 ，实际核对代码
```
