---
type: domain-base
sub-type: repository-index
module: contract
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/contract/mapper/
owner: cordys-crm
verify-required: true
verify-note: 表名/字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-contract.md
---

# 合同模块 存储索引（repository-contract）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀 |
|---|---|---|
| **ExtContractMapper** | 合同主档 CRUD/分页 JOIN 扩展字段 | crm_contract |
| **ExtContractFieldMapper / Blob** | 合同扩展字段 | crm_contract_field / _blob |
| **ExtContractSnapshotMapper** | 合同快照（每次阶段推进保存） | crm_contract_snapshot |
| **ExtContractStageMapper**（若独立） | 合同阶段配置 | crm_contract_stage |
| **ExtContractStageHistoryMapper** | 合同阶段变更历史 | crm_contract_stage_history |
| **ExtBusinessTitleMapper** | 工商抬头 CRUD | crm_business_title |
| **ExtContractInvoiceMapper** | 发票记录 | crm_contract_invoice（或 crm_invoice，核对） |
| **ExtContractPaymentPlanMapper** | 收款计划 | crm_contract_payment_plan |
| **ExtContractPaymentRecordMapper** | 收款记录 | crm_contract_payment_record |

XML 路径：`resources/mapper/contract/Ext{Xxx}Mapper.xml`

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_contract_tables.sql`

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 合同阶段对比（快照 2 和 快照 3 差异） | ExtContractSnapshotMapper.selectByContractId 按版本号取 |
| 某合同 还剩多少未收款 / 收款进度 SQL | ExtContractPaymentPlanMapper.xml JOIN payment_record SUM |
| 开票统计（某月开了多少票） | ExtContractInvoiceMapper.xml selectInvoiceSummary |
| 工商抬头 审批通过前/后的 两个版本快照 | BusinessTitle 自身是否存版本表？或走 operation_log diff，核对实现 |
