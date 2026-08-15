---
type: state
status: official
evidence: 前端枚举（clueEnum/customerEnum/opportunityEnum/contractEnum）+ CODE_WIKI.md
owner: cordys-crm
verify-required: true
verify-note: 实际枚举值必须回到代码仓库核对：frontend/packages/lib-shared/enums/*.ts 以及 backend domain 类
updated: 2026-08-11
---

# Cordys CRM · 通用状态定义

> 本文件定义各模块核心状态的语义与流转条件。
> 
> ⚠️ **强制核对要求**：实际编码时，状态的**枚举名、编码、顺序**必须回到以下位置核对当前代码：
> - 前端: `frontend/packages/lib-shared/enums/{module}Enum.ts`
> - 前端配置: `backend/crm/src/main/resources/form/field.json`
> - 后端 Domain: `backend/crm/src/main/java/cn/cordys/crm/{module}/domain/*.java`

---

## 一、线索状态 ClueStatus

| 枚举值 | 中文名 | 含义 | 流转进入条件 | 可流转至 |
|---|---|---|---|---|
| `NEW` | 新建 | 线索刚录入或刚分配，尚未开始跟进 | 线索创建 / 线索池分配 / 领取 | FOLLOWING / INTERESTED |
| `FOLLOWING` | 跟进中 | 正在跟进沟通 | 首次跟进记录写入 / 手动切换 | INTERESTED |
| `INTERESTED` | 感兴趣 | 客户表现出明确兴趣，进入转客户阶段 | 跟进中标记感兴趣 / 线索评分达标 | （转客户后线索归档） |

**终态说明**：线索无显式"已完成"终态，通过「转客户」动作完成生命周期，转后线索被标记为已转化并归档。

**后端定位入口**：
- `cn.cordys.crm.clue.domain.Clue` 类中 `status` 字段
- 状态变更逻辑在 `ClueService` 相关方法

---

## 二、客户状态

| 状态方向 | 说明 |
|---|---|
| **常规状态** | 客户默认为正常状态，无专门的状态字段。客户的"活跃性"通过最近跟进时间间接判断。 |
| **回收状态** | 超过设置天数未跟进的客户 → 自动回收到公海（CustomerPool），从负责人个人列表中消失 |
| **移交** | 客户负责人变更（旧负责人→新负责人） |
| **协作加入/退出** | 协作人列表变化，不影响客户本身状态 |

**后端定位入口**：
- `cn.cordys.crm.customer.domain.Customer`
- `cn.cordys.crm.customer.service.CustomerService`（回收/移交/协作）

---

## 三、跟进计划状态 FollowPlanStatus

| 枚举值 | 中文名 | 含义 |
|---|---|---|
| `PREPARED` | 待执行 | 计划创建，尚未到开始时间或尚未开始 |
| `UNDERWAY` | 进行中 | 到了计划时间，正在执行 |
| `COMPLETED` | 已完成 | 已写入跟进记录，计划闭环 |
| `CANCELLED` | 已取消 | 手动取消该计划 |

---

## 四、商机状态与阶段 Opportunity

### 4.1 商机阶段（OpportunityStage，可配置）

默认阶段顺序（非枚举，来自数据库配置 `opportunity_stage_config`）：

| 顺序 | 阶段名 | 典型赢单率 | 说明 |
|---|---|---|---|
| 1 | 初步接触 | 10% | 刚建立联系，确认对方有潜在需求 |
| 2 | 需求确认 | 30% | 明确客户需求、预算、决策链 |
| 3 | 方案制定 | 50% | 提供解决方案/报价初稿 |
| 4 | 商务谈判 | 70% | 价格/条款谈判，进入决策阶段 |
| 5 | 合同签订 | 90% | 条款确认，走合同签署流程 |

⚠️ 实际阶段配置需查:
- 后端配置表: `opportunity_stage_config`（Flyway 迁移或配置页面）
- 前端配置: `frontend/packages/web/src/config/opportunity.ts`

### 4.2 商机结果

在阶段推进至最后或中途时，标记商机结果：
- **赢单** → 进入赢单审批（可选）→ 自动创建合同草稿
- **输单** → 输单原因必填 → 输单审批（可选）→ 归档

---

## 五、合同状态 ContractStatus

| 枚举值 | 中文名 | 含义 | 可流转至 |
|---|---|---|---|
| `PENDING_SIGNING` | 待签署 | 合同已创建/已审批，等待双方签署 | SIGNED（签署完成） / VOID（作废） |
| `SIGNED` | 已签署 | 双方已签署，合同生效 | CHANGE（变更） / IN_PROGRESS（开始履行） / VOID |
| `CHANGE` | 变更中 | 已签署合同进入变更流程 | SIGNED（变更完成） / IN_PROGRESS |
| `IN_PROGRESS` | 履行中 | 正在执行（订单、收款、开票进行中） | COMPLETED_PERFORMANCE / VOID |
| `COMPLETED_PERFORMANCE` | 履行完毕 | 订单/收款/开票全部闭环 | ARCHIVED |
| `VOID` | 作废 | 合同取消或误操作作废 | （终态，不可逆转） |
| `ARCHIVED` | 已归档 | 合同完结合档 | （终态） |

---

## 六、收款计划状态 ContractPaymentPlan

| 枚举值 | 中文名 | 含义 |
|---|---|---|
| `PENDING` | 未完成 | 收款计划创建，未到账或仅部分到账 |
| `PARTIALLY_COMPLETED` | 部分完成 | 已有部分收款核销，但金额未满 |
| `COMPLETED` | 已完成 | 全部计划金额已收齐 |

---

## 七、工商抬头审批状态 ContractBusinessTitleStatus

| 枚举值 | 中文名 | 含义 |
|---|---|---|
| `UNAPPROVED` | 未通过 | 新建/驳回 |
| `APPROVING` | 提审中 | 审批流运行中 |
| `APPROVED` | 通过 | 审批通过，可用于开票 |
| `REVOKED` | 撤销 | 发起人撤销审批 |

---

## 八、审批流通用状态 ApprovalInstance

| 状态 | 含义 |
|---|---|
| DRAFT | 草稿（未提交审批） |
| PROCESSING | 审批中（节点流转中） |
| APPROVED | 审批通过 |
| REJECTED | 审批驳回 |
| REVOKED | 发起人撤销 |

**节点任务状态**（ApprovalTask）：
- PENDING: 待处理
- APPROVED: 已通过
- REJECTED: 已驳回
- TRANSFERRED: 已转办
- COUNTERSIGNED: 加签处理中

---

## 九、订单阶段 OrderStage

订单阶段为**可配置项**（来自 `order_stage_config`），默认配置参考：

| 顺序 | 默认阶段 | 说明 |
|---|---|---|
| 1 | 待执行 | 订单创建，尚未启动 |
| 2 | 执行中 | 生产/备货/服务启动 |
| 3 | 已发货 | 产品已发货/服务已开始交付 |
| 4 | 已签收 | 客户已签收/确认交付 |
| 5 | 已完成 | 订单完成，进入历史 |

⚠️ 实际阶段需查:
- 后端表: `order_stage_config`
- 前端配置: `frontend/packages/web/src/config/`

---

## 十、状态流转通用约束

**原则 1: 终态不可逆**
- VOID / ARCHIVED / COMPLETED_PERFORMANCE / APPROVED(审批) 等终态一旦进入，不可逆向回到中间状态。
- 如需修正，走：作废 → 新建 或 变更流程（Contract Change）。

**原则 2: 审批中限制**
- 单据处于审批流 PROCESSING 状态时：禁止修改关键字段（金额、对方、产品...），仅允许加签/转办/撤销。

**原则 3: 操作日志**
- 所有状态变更必须写入操作日志（`@OperationLog` 切面自动处理）。
- AI 修改状态流转代码时，必须确认：状态变化前后均有操作日志记录。

**原则 4: 回到代码核对**
- 以上状态枚举仅作语义参考。实际编码前必须回到前端枚举 + 后端 Domain 类核对枚举名、类型、注解。
