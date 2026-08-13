---
type: domain-base
sub-type: api-index
module: contract
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/contract/controller 目录
owner: cordys-crm
verify-required: true
verify-note: 开票接口/收款计划/工商抬头审批 参数 易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/contract.md
  - knowledge/applications/backend/domain/base/domain-contract.md
  - knowledge/applications/backend/domain/base/repository-contract.md
---

# 合同模块 API 索引（api-contract）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **ContractController** | 合同主档 CRUD、合同阶段推进、合同作废/撤销作废、合同快照查看、合同来源（商机/报价）创建、批量编辑、导入导出入口 | ContractService + ContractStageService + ContractExportService | `CONTRACT:ADD/EDIT/DELETE/VIEW/VOID/ADVANCE_STAGE/BATCH_EDIT/EXPORT` |
| **ContractInvoiceController** | 合同发票：开票申请、发票列表、发票详情、红字冲回（作废发票）、收款计划 CRUD、收款记录录入 | ContractInvoiceService（发票+计划+记录统一封装） | `CONTRACT:INVOICE_ADD/EDIT/DELETE/VOID + PAYMENT_PLAN_* + PAYMENT_RECORD_*` |
| **BusinessTitleController** | 工商抬头 CRUD、抬头审批提交/撤回、抬头启用/停用、抬头字段配置 | BusinessTitleService | `CONTRACT:BUSINESS_TITLE_ADD/EDIT/DELETE/APPROVAL_MANAGE` |

> 注：**收款计划/收款记录**接口若在独立 Controller（核对代码），在上述表格拆分或合并到 ContractInvoiceController，一切以实际代码为准，易变项不写死。

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 合同本身 / 阶段 / 作废 / 快照 / 导入导出 | ContractController |
| 开票 / 发票 / 红字冲回 / 收款计划 / 收款记录 | ContractInvoiceController（或拆分为 ContractPaymentController，核对代码） |
| 工商抬头 / 抬头审批 / 抬头维护 | BusinessTitleController |

路径前缀（核对 @RequestMapping）：
```
/api/contract/*          → 合同主档
/api/contract-invoice/*  → 发票/收款
/api/business-title/*    → 工商抬头
```
