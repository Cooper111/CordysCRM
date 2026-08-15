---
type: domain-product
app: backend
module: opportunity
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/opportunity 下 Opportunity / OpportunityQuotation / OpportunityStage / OpportunityRule 四套子能力（Controller+Service+Mapper）
owner: cordys-crm
verify-required: true
verify-note: 阶段枚举/报价字段/规则表达式/审批配置必须回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#商机/报价单/商机阶段/赢单/输单
  - knowledge/main/core-process.md#4-商机跟进链路
  - knowledge/main/state-defs.md#四商机阶段
  - knowledge/main/business-rules.md#审批触发#4商机输单回收/自动阶段
  - knowledge/applications/backend/domain/base/api-opportunity.md
  - knowledge/applications/backend/domain/base/domain-opportunity.md
  - knowledge/applications/backend/domain/base/repository-opportunity.md
---

# 商机模块 · 产品能力

---

## 一、能力概述

从"发现销售机会→报价→多阶段推进→赢单/输单→赢单后驱动合同/订单"的销售过程管理。包含商机主档、报价单、阶段配置与推进、商机规则引擎（自动阶段/赢输单回收）、视图筛选。商机是**连接客户（客户）到合同/订单（回款）的关键桥梁**。

主干流程：
```
新建商机（关联客户） → 录入首阶段 → 报价 → 按阶段推进（商机规则自动跳阶段）
                                              ↓ 赢单             ↓ 输单
                                 触发合同起草建议      记录输单原因 → 可能触发商机回收回负责人/公海
```

---

## 二、核心流程（ASCII 流程图）

```
 ┌──────────────────────── 新建入口 ────────────────────────┐
 │ ① 手工新建商机  ② 客户详情下新建  ③ 赢单/输单线索转化     │
 └──────────────────────────────┬───────────────────────────┘
                                ▼
                     【商机创建 + 初始阶段设置】
            Opportunity + CustomerId + 商机字段 + OpportunityField
                                │
                                ▼
                     ┌── 初始阶段（stage_id=第一阶段配置序号）
                     │    Stage 来源：OpportunityStageConfig（可排序）
                     │
         ┌───────────┴─────────────────────────────────────────┐
         ▼                                                     ▼
  【报价单 Quotation】（0..N 份，可多版本）                【阶段推进】
     ├ 报价明细关联 Product                              OpportunityRuleService：
     ├ 税率/折扣/总金额自动计算                          if 满足某阶段"进入条件表达式"
     └ 报价打印 PDF（导出）                                   → 自动跳阶段
                                                                    │
                          OpportunityStageController
                          手动改阶段 + 写阶段变更记录
                                            │
                          ┌─────────────────┼────────────────┐
                          ▼                 ▼                ▼
                      【赢单】            【输单】       【进行中】
                   赢单原因必填      输单原因必填 +    按阶段/规则推进
                   状态 = WON           状态 = LOST
                          │                 │
                          ▼                 ▼
                触发商机转合同       OpportunityRuleListener
                   入口（前端弹      按回收规则可能：
                    "基于本商机       ① 回退阶段 ② 回收给负责人 ③ 退回公海
                     起草合同"按钮）
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口（定位类+方法） |
|---|---|---|---|
| 商机创建 | 手工新建 / 从客户新建 | 必须关联 customerId（有效客户）→ 写 Opportunity + 初始阶段 + 字段 + 操作日志 | OpportunityService.add |
| 报价单创建/修改 | 商机详情页→报价 Tab | 报价明细：每条明细 productId + qty + price → 自动算小计 → 报价版本号递增 → 写 Snapshot | OpportunityQuotationService.add / update |
| 阶段推进 | 手动点击下阶段 / 规则自动触发 | 校验：阶段切换条件（按业务自定义 OpportunityStageConfig.stageAdvancedConfig）→ 更新 opportunity.stageId → 写阶段变更日志 → 通知负责人 | OpportunityStageService.advance / OpportunityRuleService.evaluate |
| 赢单/输单 | 点击赢/输按钮 | 必填赢单原因 or 输单原因字段；状态变为 WON / LOST；赢单后"商机转合同"按钮高亮（前端） | OpportunityService.win / lose + OpportunityStageController 触发输单回收 |
| 商机回收（规则触发） | 满足 OpportunityRule 的回收条件 | OpportunityRuleListener → 判断是否满足规则 → 按规则类型：回退阶段 / 退回负责人 / 退回公海 | OpportunityRuleListener（system/job/listener + OpportunityRuleService.execute） |
| 商机导入/导出 | Excel 导入导出 | 导出：OpportunityExportService（分页+流式，大导出异步）；导入：按 system/Import 流程校验字段 | OpportunityExportService + ImportType.OPPORTUNITY |

---

## 三、核心业务规则

| 规则 ID | 规则描述 | 例外 |
|---|---|---|
| R1 | **商机必须绑定有效客户**：if (customerId 不存在 OR Customer.status=已删除/已回收) then 拒绝创建 | 管理员创建草稿商机可绑定"临时客户占位"（需后续补正式客户） |
| R2 | **阶段只能前进，不能回退（默认配置）**：if (目标阶段 < 当前阶段序号) then 默认拒绝 | 开关"阶段允许回退"在 OpportunityStageConfig 打开；或管理员强制回退（写操作日志） |
| R3 | **赢/输 状态后不可修改关键信息**：if (status=WON or LOST) then 禁止修改金额 / 客户 / 阶段；只允许新增跟进记录/评论 | 管理员强制撤销赢/输单（写风险操作日志） |
| R4 | **报价版本**：if (该商机下已有生效报价单) then 编辑"报价字段"会自动产生新版本，旧版本可对比不可编辑 | 管理员可直接删除报价（需记录原因） |
| R5 | **回收规则触发**：if (满足一个以上回收规则) then 按优先级最高的规则执行（不重复触发） | 规则命中后写"商机回收日志"（操作日志） |
| R6 | **商机导出权限**：无 OPPORTUNITY:EXPORT 节点 → 403 CsPermissionAspect 拦截 | 导出字段同样受"模块字段可见权限"约束 |
| R7 | **商机金额汇总统计**：Dashboard 商机金额报表 = 按阶段聚合（排除 LOST、排除作废、WON 算入"已赢单"看板） | 组织参数"商机金额统计口径"可配置：是/否包含 WON/草稿 |
| R8 | **阶段高级配置表达式**：阶段切换条件/商机规则表达式 → 采用 RuleConditionDTO（JSON 规则树），由 ConditionFilterUtils 解析 → 不允许直接写脚本/SQL | 条件引擎统一入口，禁止写原生 SQL 进规则配置表 |

---

## 四、状态/枚举（定位入口）

| 语义 | 枚举/类名 入口 | 代码路径 |
|---|---|---|
| 商机状态（新建/跟进中/赢单/输单/作废） | `Opportunity.status` 字段 + 字典 DictModule.OPPORTUNITY_STATUS | crm.opportunity.domain.Opportunity + system.domain.Dict |
| 商机阶段（阶段 ID 排序） | `OpportunityStageConfig.sort` 排序 + stageId 外键 | crm.opportunity.domain.OpportunityStageConfig |
| 阶段高级配置类型 | `StageAdvancedConfig` 表（system.domain） + OpportunityStage.advancedConfId | system.domain.StageAdvancedConfig |
| 回收规则类型（回退/回负责人/回公海） | `OpportunityRule` 表字段 type + 对应枚举 | crm.opportunity.domain.OpportunityRule |
| 表单 Key | `FormKey.OPPORTUNITY / OPPORTUNITY_QUOTATION` | common.constants.FormKey |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **customer** | opportunity ↔ customer | 商机必须关联 customer.customerId；客户删除/反转化前校验：客户下无进行中商机（非 WON/非 LOST） |
| **product（报价明细）** | opportunity ↔ product | 报价单明细引用 Product.productId + Price.priceId（从 ProductService 查询最新价）；删除产品时校验未被引用 |
| **contract（合同）** | opportunity → contract | 合同可"来源商机"关联 Contract.opportunityId；赢单后前端推荐"基于本商机起草合同"（基于报价明细自动生成合同行） |
| **follow（跟进）** | opportunity → follow | OpportunityFollowPlanController / OpportunityFollowRecordController 调 follow.*Service，resourceType=OPPORTUNITY + resourceId=opportunity.id / quotationId |
| **approval** | opportunity ← approval | 阶段推进 / 赢单输单可启用审批；审批通过后执行推进 / 赢单 / 输单动作（ApprovalActionService 后回调） |
| **dashboard（报表）** | dashboard → opportunity | Dashboard 统计：按阶段聚合 opportunity.amount / 赢输单统计 | DashboardService.countOpportunity* |
| **system（规则/视图/日志）** | opportunity → system | OpportunityRuleListener 定时任务；视图 OpportunityUserViewController（UserViewService）；OpportunityLogService + QuotationalLogService |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口类 |
|---|---|---|
| **商机金额小数点精度问题**（报价累加 vs 总计不一致） | 报价单总计 = 前端提交 totalAmount，后端**再按明细重算一遍**对比；差异 > 0.01 拒绝 | OpportunityQuotationService.add 末尾 |
| **阶段配置变更后老商机"卡阶段"** | 阶段配置删除/调整顺序后 → 老商机 stageId 变成无效：采用"保持原 stageId + 前端灰色显示（已废弃阶段）+ 引导切换"策略 | OpportunityStageService 查配置时容错 |
| **报价明细关联的产品已删除** | 删除产品前：ProductService 检查"是否有报价明细引用本产品" → 如有则拒绝删除，提示"请先清理报价" | ProductService.delete precheck + ExtOpportunityQuotationMapper.countRefProduct |
| **赢单后无法撤销（误操作）** | 管理员操作"撤销赢单"：状态回到上一阶段；需记录操作日志撤销原因；合同若已生成则提醒"合同仍在，需手动作废或解除关联" | OpportunityService.cancelWin 扩展（如已有）+ OperationLog |
| **商机规则命中频率过高（抖动）** | OpportunityRuleListener 每次只处理 N 条 + 单条商机 1 天内最多被回收 1 次（写去重标记） | OpportunityRuleListener 执行流程的 N 条 + 去重 |
| **大量商机（100W+）Dashboard 报表慢** | Dashboard 统计走 DashboardService → 查统计中间表（如有的话）或按日/周分区查询；实时 Dashboard 建议走 DataEase 同步 | DashboardService 对应统计方法 |
