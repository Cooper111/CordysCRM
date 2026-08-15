---
type: domain-base
sub-type: repository-index
module: opportunity
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/opportunity/mapper/
owner: cordys-crm
verify-required: true
verify-note: 表名/字段 易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-opportunity.md
---

# 商机模块 存储索引（repository-opportunity）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀 |
|---|---|---|
| **ExtOpportunityMapper** | 商机主档 CRUD/分页 JOIN 扩展字段 | crm_opportunity |
| **ExtOpportunityFieldMapper / Blob** | 商机扩展字段 | crm_opportunity_field / _blob |
| **ExtOpportunityStageMapper** | 商机阶段配置 | crm_opportunity_stage（或 stage_config，核对） |
| **ExtOpportunityStageHistoryMapper** | 阶段变更历史 | crm_opportunity_stage_history |
| **ExtOpportunityQuotationMapper** | 报价单主档 | crm_opportunity_quotation（或 crm_quotation，核对 @TableName） |
| **ExtOpportunityQuotationItemMapper** | 报价单明细 | crm_opportunity_quotation_item |
| **ExtOpportunityRuleMapper** | 自动规则配置 | crm_opportunity_rule |
| **ExtOpportunityWinLoseRecordMapper**（若独立） | 赢/输单记录 | crm_opportunity_win_lose_record |

XML 路径：`resources/mapper/opportunity/Ext{Xxx}Mapper.xml`

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_opportunity_tables.sql`（具体日期不同，不允许修改已运行脚本）

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 商机分页列表/按阶段/按赢率过滤 SQL | ExtOpportunityMapper.xml |
| 报价行项批量保存 | ExtOpportunityQuotationItemMapper.batchInsert |
| 商机阶段停留时长统计 | ExtOpportunityStageHistoryMapper.xml |
| 商机自动规则扫描（定时 Job 找符合条件的商机） | ExtOpportunityMapper.xml selectForRuleMatch 或对应方法 |
