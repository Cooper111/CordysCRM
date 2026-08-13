---
type: domain-base
sub-type: domain-index
module: contract
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/contract/ 目录
owner: cordys-crm
verify-required: true
verify-note: 发票/收款阶段/工商抬头字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/contract.md
  - knowledge/applications/backend/domain/base/api-contract.md
---

# 合同模块 领域模型索引（domain-contract）

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **Contract** | 合同主档：合同编号、名称、关联客户、关联商机/报价、签订日期、生效日期、到期日期、合同金额、负责人、阶段(draft/signing/executing/completed/void)、是否作废 | crm_contract | ContractExtMapper | Y（ContractField + ContractFieldBlob） |
| **ContractField** | 合同扩展字段行 | crm_contract_field | ContractFieldExtMapper | N |
| **ContractFieldBlob** | 合同扩展 BLOB | crm_contract_field_blob | ContractFieldBlobExtMapper | N |
| **ContractSnapshot** | 合同快照（阶段推进 时 或 审批通过时/作废前 自动保存 合同 JSON 快照） | crm_contract_snapshot | ContractSnapshotExtMapper | N |
| **ContractStage** | 合同阶段配置（管理员）：阶段 CRUD、排序、进入条件 JSON（通常合同阶段数较少） | crm_contract_stage（若独立表） | ContractStageExtMapper | N |
| **ContractStageHistory** | 合同阶段变更历史 | crm_contract_stage_history | ContractStageHistoryExtMapper | N |
| **BusinessTitle** | 工商抬头（我方/客户方开票信息）：抬头名称、纳税人识别号、开户行、银行账号、地址电话、启用状态、审批状态 | crm_business_title | BusinessTitleExtMapper | N |
| **ContractInvoice** | 开票记录：发票号、开票日期、抬头 id、合同 id、开票金额、税额、税率、发票状态（正常/红冲/作废） | crm_contract_invoice | ContractInvoiceExtMapper | N |
| **ContractPaymentPlan** | 收款计划：期次、计划收款日期、计划收款金额、实际收款金额（累计 = ContractPaymentRecord 求和）、收款条件说明 | crm_contract_payment_plan | ContractPaymentPlanExtMapper | N |
| **ContractPaymentRecord** | 收款记录：到账日期、到账金额、关联计划 id、付款账户、凭证号、备注 | crm_contract_payment_record | ContractPaymentRecordExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.contract.dto.*`
- export：`cn.cordys.crm.contract.dto.export.*`

---

## 二、定位技巧

| 关键词 | 类 |
|---|---|
| 合同本身 / 合同编号 / 金额 / 阶段 / 作废 | Contract |
| 合同自定义字段 | ContractField + Blob |
| 合同历史版本 / 快照对比 / 审计追溯 | ContractSnapshot |
| 合同阶段配置 / 阶段推进记录 | ContractStage + ContractStageHistory |
| 开票信息 / 工商抬头 / 抬头审批 | BusinessTitle |
| 发票 / 红冲 / 开票记录 | ContractInvoice |
| 收款计划（分几期） / 收款 提醒 | ContractPaymentPlan |
| 到账记录 / 已收了多少钱 | ContractPaymentRecord |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **ContractStatusEnum / ContractStageEnum** | 合同状态/阶段 |
| **ContractVoidReasonEnum** | 合同作废原因（常走字典，核对） |
| **InvoiceStatusEnum** | 发票状态（正常/红冲/作废） |
| **BusinessTitleApprovalStatusEnum** | 工商抬头审批状态 |
