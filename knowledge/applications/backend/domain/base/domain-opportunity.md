---
type: domain-base
sub-type: domain-index
module: opportunity
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/opportunity/ 目录
owner: cordys-crm
verify-required: true
verify-note: 商机阶段/赢输原因/报价字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/opportunity.md
  - knowledge/applications/backend/domain/base/api-opportunity.md
---

# 商机模块 领域模型索引（domain-opportunity）

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **Opportunity** | 商机主实体：名称、关联客户、关联线索、阶段、赢率(%)、预计成交金额、预计成交日期、负责人 | crm_opportunity | OpportunityExtMapper | Y（OpportunityField + OpportunityFieldBlob） |
| **OpportunityField** | 商机扩展字段行 | crm_opportunity_field | OpportunityFieldExtMapper | N |
| **OpportunityFieldBlob** | 商机扩展 BLOB | crm_opportunity_field_blob | OpportunityFieldBlobExtMapper | N |
| **OpportunityStage** | 商机阶段配置（管理员维护）：阶段名称、阶段排序、赢率阶梯、进入条件表达式 JSON | crm_opportunity_stage | OpportunityStageExtMapper | N |
| **OpportunityStageHistory** | 商机阶段变更历史：fromStage / toStage / operator / operateTime / 阶段停留时长(计算字段) | crm_opportunity_stage_history | OpportunityStageHistoryExtMapper | N |
| **OpportunityQuotation** | 商机报价单主档：报价编号、版本号、报价日期、有效期、是否生效版本、关联商机 id、总价、税额、折扣、状态（草稿/已确认/已作废） | crm_opportunity_quotation | OpportunityQuotationExtMapper | Y（报价字段扩展，核对 QuotationField+Blob） |
| **OpportunityQuotationItem** | 报价行项明细：报价单 id、产品 id、产品快照 JSON（保存时快照）、数量、单价、折扣率、小计、税额 | crm_opportunity_quotation_item | OpportunityQuotationItemExtMapper | N |
| **OpportunityRule** | 商机自动规则（如"30 天未推进自动回上一阶段"、"赢单后自动创建合同草稿"）：规则表达式 JSON、规则优先级、是否启用 | crm_opportunity_rule | OpportunityRuleExtMapper | N |
| **OpportunityWinLoseRecord** | 赢/输单记录：opportunityId、result(WIN/LOSE)、reasonDictId（赢/输原因字典）、description、operateTime | crm_opportunity_win_lose_record（核对） | OpportunityWinLoseRecordExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.opportunity.dto.*`
- export：`cn.cordys.crm.opportunity.dto.export.*`

---

## 二、定位技巧

| 关键词 | 类 |
|---|---|
| 商机本身 / 名称 / 阶段 / 赢率 / 金额 | Opportunity |
| 商机自定义字段 | OpportunityField + Blob |
| 阶段有哪些 / 阶段配置 / 阶段顺序 / 阶段赢率 | OpportunityStage |
| 某商机 阶段变更 轨迹 / 每个阶段待了多久 | OpportunityStageHistory |
| 报价单 / 多版本报价 / 报价明细 行 | OpportunityQuotation + OpportunityQuotationItem |
| 商机 自动阶段规则 / 自动 赢 输单 回收 | OpportunityRule |
| 赢/输单 原因 / 赢输 分析 | OpportunityWinLoseRecord（若独立） |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **OpportunityStatusEnum** 或 **OpportunityStageEnum** | 商机状态/阶段枚举（通常阶段走配置表而非枚举，核对） |
| **OpportunityResultEnum** | 结果枚举（WIN/LOSE/IN_PROGRESS…） |
| **QuotationStatusEnum** | 报价单状态（草稿/已确认/已作废/已过期…） |
