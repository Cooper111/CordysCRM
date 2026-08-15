---
type: domain-base
sub-type: api-index
module: opportunity
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/opportunity/controller 目录
owner: cordys-crm
verify-required: true
verify-note: 易变项：阶段推进接口/赢输单参数/报价多版本接口 必须核对代码最新实现
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/opportunity.md
  - knowledge/applications/backend/domain/base/domain-opportunity.md
  - knowledge/applications/backend/domain/base/repository-opportunity.md
---

# 商机模块 API 索引（api-opportunity）

> 定位入口，非接口文档。易变项（赢/输请求结构、报价明细 DTO）核对代码。

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **OpportunityController** | 商机主档 CRUD、阶段推进、赢单/输单、商机移交、批量编辑、导入导出入口、商机统计图表 | OpportunityService + OpportunityExportService | `OPPORTUNITY:ADD/EDIT/DELETE/VIEW/TRANSFER/BATCH_EDIT/WIN/LOSE/EXPORT` |
| **OpportunityQuotationController** | 商机报价单 CRUD、报价版本（多版本切换/对比）、报价明细、导出/打印入口 | OpportunityQuotationService + OpportunityQuotationFieldService | `OPPORTUNITY:QUOTATION_ADD/EDIT/DELETE/VIEW` |
| **OpportunityStageController** | 商机阶段配置管理（管理员）：阶段 CRUD、排序、阶段高级配置（进入条件表达式）、阶段推进校验 | OpportunityStageService | `OPPORTUNITY:STAGE_MANAGE` |
| **OpportunityRuleController** | 商机规则管理（管理员）：自动阶段/赢单输单回收规则 CRUD、规则优先级、规则测试 | OpportunityRuleService | `OPPORTUNITY:RULE_MANAGE` |
| **OpportunityUserViewController** | 商机列表筛选视图保存 | UserViewService（system） | `OPPORTUNITY:VIEW + USER_VIEW:*` |
| **OpportunityQuotationUserViewController** | 报价单列表筛选视图保存 | UserViewService（system） | `OPPORTUNITY:QUOTATION_VIEW + USER_VIEW:*` |
| **OpportunityFollowPlanController** | 商机→跟进计划（代理 follow resourceType=OPPORTUNITY） | FollowUpPlanService | `OPPORTUNITY:FOLLOW_PLAN_*` |
| **OpportunityFollowRecordController** | 商机→跟进记录（代理 follow resourceType=OPPORTUNITY） | FollowUpRecordService | `OPPORTUNITY:FOLLOW_RECORD_*` |

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 商机本身 CRUD / 赢单 / 输单 / 阶段推进 | OpportunityController |
| 报价单（多版本）/ 报价明细 | OpportunityQuotationController |
| 管理员→阶段配置（有多少个阶段、顺序） | OpportunityStageController |
| 管理员→商机自动规则（自动阶段/回收） | OpportunityRuleController |
| 商机 / 报价 保存筛选视图 | OpportunityUserViewController / OpportunityQuotationUserViewController |
| 商机 Tab：跟进计划 / 跟进记录 | OpportunityFollowPlanController / OpportunityFollowRecordController |

路径前缀（核对 @RequestMapping）：
```
/api/opportunity/*               → 商机主档
/api/opportunity-quotation/*     → 报价
/api/opportunity-stage/*         → 阶段配置
/api/opportunity-rule/*          → 自动规则
/api/opportunity-view/*          → 商机视图
/api/opportunity-quotation-view/* → 报价视图
/api/opportunity-follow-plan/* / -record/* → 跟进
```
