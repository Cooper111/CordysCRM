---
type: domain-product
app: backend
module: contract
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/contract 下：Contract（主档+阶段+快照+字段）、BusinessTitle（工商抬头）、ContractInvoice（发票+开票+收款计划+收款记录+快照+字段）
owner: cordys-crm
verify-required: true
verify-note: 合同审批触发条件、收款金额计算、快照时机、工商抬头审批状态必须回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#合同/工商抬头/开票/收款计划/收款记录
  - knowledge/main/core-process.md#5-合同-收款链路
  - knowledge/main/state-defs.md#五合同状态+#六收款计划状态+#七工商抬头审批状态
  - knowledge/main/business-rules.md#审批触发规则#4-合同/工商抬头审批
  - knowledge/applications/backend/domain/base/api-contract.md
  - knowledge/applications/backend/domain/base/domain-contract.md
  - knowledge/applications/backend/domain/base/repository-contract.md
---

# 合同模块 · 产品能力

---

## 一、能力概述

管理销售合同从起草→审批→阶段推进→完结→作废的完整生命周期，包含：合同主档（字段/快照）、合同阶段配置、工商抬头（BusinessTitle，合同开票信息主档+审批）、发票（Invoice）、收款计划（PaymentPlan）、收款记录（PaymentRecord）。合同是**订单的前置、回款的依据，直接关联财务数据**。

主干流程：
```
合同起草（关联客户/来源商机/来源报价） → 阶段推进（配置化） → 合同审批（可选）
       ↓ 审批通过
  发票 + 收款计划
    ↓       ↓
  开票   分期收款记录（部分回款/全额回款）
       ↓
   合同快照（各关键节点自动打快照）
       ↓
  完结 / 作废（作废可撤销）
```

---

## 二、核心流程（ASCII 流程图）

```
                ┌──────────── 合同 3 种创建入口 ───────────┐
                │ ① 起草（手动） ② 来源商机 ③ 来源报价单    │
                └─────────────────────┬───────────────────┘
                                      ▼
                        ContractService.add（合同主档 + 字段）
                   合同 → customerId + （可选）opportunityId + quotationId
                                      │
                                      ▼
                          【合同阶段 ContractStageService】
                   stageId ← ContractStageConfig（可配置，阶段按 sort 推进）
                                      │
                    ┌─────────────────┴─────────────────────┐
                    ▼                                       ▼
           【合同审批（可选）】                       【工商抬头 BusinessTitle】
           @HitApproval(ApprovalTypeEnum.CONTRACT)       BusinessTitleService
           命中 → 提交审批流 → 审批通过                      ↑
                    │                                     │ 抬头用于开票
                    ▼                                     ▼
           stage推进 + 快照(ContractSnapshot)   ContractInvoice（发票）
                    │                             ├── 关联合同ID + 抬头ID
                    │                             ├── 开票 → 发票字段 + Snapshot
                    ▼                             └── 作废发票（红字冲回）
          【收款计划 ContractPaymentPlan】
                    ├ 分期：plan 1..N 期，每期金额 + 到期日
                    └ 状态：未到期/已逾期/已完成
                              │
                              ▼
                   【收款记录 ContractPaymentRecord】
                    关联 planId + 本次收款金额 + 凭证附件
                    → 回填 plan.实收金额 + 状态（已完成=100%）
                              │
                              ▼
                  （全部计划100%回款）→ 合同自动状态=已完成（或手动完结）
                       ↓ 业务异常
               【合同作废 ContractService.void】
                    写入作废原因 + 保留快照（审计可追溯）
                       ↓ 误操作撤销作废
               ContractService.cancelVoid（需权限）
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 合同创建（手动/来源商机/来源报价） | 合同页"新建合同"按钮 | 字段校验→关联有效性检查→写 Contract 主表 + ContractField + 初始阶段→操作日志→（自动打首份快照） | ContractService.add + ContractStageService.init |
| 合同阶段推进 | 点击"下一阶段"按钮 | 阶段配置→检查前置条件（如"必须先有收款计划"）→更新stageId→打快照+写操作日志 | ContractStageService.advanceStage(ContractStageRequest) |
| 提交合同审批 | 组织配置"合同审批"=开 | @HitApproval 注解切面 → 命中审批类型 CONTRACT → 生成审批实例 ApprovalInstance → 状态=审批中→审批通过回调 stage+快照 | approval.aspect.HitApprovalAspect + BusinessTitleService |
| 工商抬头管理 | 系统管理→工商抬头页（或合同页选择/新增） | 抬头：BusinessTitle（公司名/税号/开户行/地址/...）→ 需审批（BusinessTitle审批状态）→审批通过后才能选作开票抬头 | BusinessTitleService.add + 审批（ApprovalTypeEnum.BUSINESS_TITLE） |
| 发票（开票） | 合同详情→发票 Tab→"开票" | 选开票抬头（已审批通过）→写 ContractInvoice + ContractInvoiceField + ContractInvoiceSnapshot（快照）→开票记录不可编辑（如需修改走红字冲回+新发票） | ContractInvoiceService.invoice + ContractInvoiceService.voidInvoice（红字） |
| 收款计划创建/编辑 | 合同详情→收款计划 Tab→新增计划 | 分期计划 N 条：每条 planAmount + dueDate → 计划总金额必须 == 合同含税金额（容差 0.01）否则拒绝 | ContractService（计划写入在 ContractService 或 PaymentPlanService） |
| 收款记录录入 | 财务收到回款→收款计划→录入记录 | 收款记录：关联 planId + amount + 凭证附件 + 收款日期 → 回填 plan.receivedAmount + 自动更新 plan 状态（未到期/已逾期/已完成） | ContractInvoiceService / ContractService.addPaymentRecord（具体名回代码核对） |
| 合同作废 / 撤销作废 | 合同作废按钮 / 管理员撤销 | 作废：写作废原因 voidReason + 合同状态=VOID → 快照一份 → 关联发票不可开票；撤销：状态回滚+操作日志（需权限 CONTRACT:VOID_CANCEL 节点） | ContractService.voidContract + cancelVoid |
| 合同快照 | 创建/阶段推进/审批通过/作废 等关键节点 | 自动生成 ContractSnapshot：字段值 JSON 序列化+版本号 → 可用于"合同对比"、"历史版本查看" | ContractSnapshotService（或由 ContractService 代理） |

---

## 三、核心业务规则

| 规则 ID | 规则描述（if/then） | 例外 |
|---|---|---|
| R1 | **合同金额 vs 收款计划**：if (SUM(PaymentPlan.planAmount) - 合同含税金额的绝对值 > 0.01) then 拒绝保存计划 | 合同"无税框架合同"允许分阶段签金额；可填 0 计划金额由财务后续拆分 |
| R2 | **分期收款记录不能超过计划金额**：if (SUM(PaymentRecord.amount) for a plan > plan.planAmount) then 拒绝录入；允许部分收款 → 对应计划状态"部分完成" | 管理员可强制超额（如含滞纳金/利息）→ 必须记录"超额原因"在操作日志 |
| R3 | **开票抬头必须审批通过**：if (BusinessTitle.approvalStatus != APPROVED) then 拒绝用该抬头开发票 | 草稿合同可先选"未审批抬头"用于合同起草预览，正式开票前自动强校验 |
| R4 | **发票不可编辑（会计合规）**：if (ContractInvoice.status=已开票) then 禁止 UPDATE；修改方式：红字冲回（voidInvoice，写一张负金额发票）+ 重开一张新发票 | 管理员禁止绕过，强合规场景 |
| R5 | **作废合同下游约束**：if (合同状态=作废) then 禁止新增发票/收款计划/收款记录；已有发票/计划可查看不可删除（财务留痕） | 合同撤销作废后，下游写动作恢复 |
| R6 | **工商抬头唯一性**：同组织 if (BusinessTitle.enterpriseName + taxNo 重复) then 拒绝新增 | 合并场景：提供"合并抬头"工具（未实现则管理员手工处理） |
| R7 | **审批触发开关**：合同审批 / 工商抬头审批 → 由 OrganizationConfig（组织配置）下对应开关控制 on/off，**不是默认开** | 超管可针对某合同类型临时跳过（审批配置"白名单"） |
| R8 | **合同快照时机**：创建、阶段变更、审批通过、审批驳回、作废、撤销作废 6 个节点必须打快照；其它字段变更由操作日志对比 diff 即可 | — |

---

## 四、状态/枚举（语义+定位入口）

| 语义 | 枚举/类名入口 | 代码路径 |
|---|---|---|
| 合同阶段（阶段配置表） | `ContractStageConfig` + sort 排序；合同实际阶段：Contract.stageId 外键 | crm.contract.domain.ContractStageConfig |
| 合同审批状态 | `ContractApprovalStatus` 枚举（草稿/审批中/已通过/已驳回） | crm.contract.constants.ContractApprovalStatus |
| 合同整体状态（进行中/已完成/作废） | `Contract.status` 字段 + 字典 | crm.contract.domain.Contract.status |
| 工商抬头审批状态 | `BusinessTitleType` 或 BusinessTitle.approvalStatus 字段 → 对应 state-defs 第七章 | crm.contract.constants.BusinessTitleType |
| 收款计划状态（未到期/已逾期/部分完成/已完成） | `ContractPaymentPlan.status` 字段 + 枚举常量 | crm.contract.domain.ContractPaymentPlan |
| 工商抬头常量（字段配置键） | `BusinessTitleConstants / SystemFieldConstants`（固定字段） | crm.contract.constants.BusinessTitleConstants |
| 表单 Key | `FormKey.CONTRACT / CONTRACT_INVOICE / BUSINESS_TITLE` | common.constants.FormKey |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **customer（客户）** | contract ↔ customer | 每合同必须关联 customerId（客户）；客户下未完结合同≠0 → 拒绝客户反转化/删除 |
| **opportunity（商机）** | contract ← opportunity（来源） | 合同来源商机 OpportunityId（可选）：来源报价单 QuotationId（可选）→ 合同明细可从报价明细一键导入 |
| **order（订单）** | contract → order | 订单可来源合同：Order.contractId；合同→订单的驱动关系（合同含"合同执行"→建订单） |
| **product（产品）** | contract → product | 合同明细/收款计划无产品引用（产品线在订单/报价单中）；如需开票明细产品则走 ContractInvoiceField |
| **approval（审批）** | contract → approval | Contract 审批（@HitApproval）、BusinessTitle 审批 → 审批通过后回调 ContractService / BusinessTitleService 写状态 |
| **system（字段/通知/快照/日志）** | contract → system | 合同字段（ModuleField）通知（NotificationConstants.CONTRACT_*）；ContractSnapshot；ContractLogService（操作日志） |
| **finance/财务（下游）** | contract → (外部) | 本系统无独立财务模块；ContractInvoice + PaymentRecord 作为财务数据入口，对接财务系统走 integration 扩展 |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口类 |
|---|---|---|
| **合同金额 + 小数点后币种精度（多币种场景）** | 所有金额统一 BigDecimal，存储 2 位小数；计算/比较一律 compareTo() != 0；不使用 double | ContractService + ContractInvoiceService 全部 BigDecimal |
| **分期 10+ 条 + 历史记录 N 条页面查询慢** | 收款计划/记录分页：PageHelper；合同详情默认展示最近 5 条，更多点击"查看全部"走分页查询 | Mapper（分页）+ PageRequest |
| **作废后误操作如何审计** | 合同作废 + 撤销作废 → 都在操作日志中记录"前后状态 + 原因 + 操作人"，同时 ContractSnapshot 快照作废前后版本 | ContractLogService + ContractSnapshot |
| **审批中被驳回 → 合同数据变更后再提** | 驳回后合同回到"草稿"或"驳回待修改"状态 → 允许修改 → 再次提交审批 → 生成**新的审批实例**（旧实例保留审批记录） | approval.instance.ApprovalInstanceService + 驳回回调 |
| **同一抬头多次开票（账期发票）** | 抬头不变，发票多张（按账期开多张 ContractInvoice）→ 通过"关联合同ID"汇总；每张发票独立打快照 | ContractInvoiceService（合同维度多张发票） |
| **收款记录里有手续费（银行到账金额≠计划金额）** | 收款记录金额 = 银行实际到账；差额部分可填"手续费字段"（如 PaymentRecord 中有 handlingFee 字段）或在备注中说明；计划完成按"到账金额>=计划金额×阈值"判断 | PaymentRecord 相关 Service（字段核对代码） |
