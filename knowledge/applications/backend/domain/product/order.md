---
type: domain-product
app: backend
module: order
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/order 下 Order / OrderStageConfig / OrderField / OrderSnapshot / OrderExportService / OrderLogService
owner: cordys-crm
verify-required: true
verify-note: 阶段配置/快照时机/字段列表需回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#订单/订单阶段
  - knowledge/main/core-process.md#6-订单执行链路
  - knowledge/main/state-defs.md#八订单阶段
  - knowledge/applications/backend/domain/base/api-order.md
  - knowledge/applications/backend/domain/base/domain-order.md
  - knowledge/applications/backend/domain/base/repository-order.md
---

# 订单模块 · 产品能力

---

## 一、能力概述

订单模块管理合同执行或直接下单的销售订单生命周期：订单创建→阶段推进→订单执行→完结。包含订单主档/订单字段/订单阶段配置/订单快照/导入导出/视图筛选。订单是**与实际交付/履约/ERP 对接**的核心节点。

主干流程：
```
订单创建（手工/来源合同/来源商机） → 阶段配置化推进 → 订单快照（关键节点）
          ↓
  订单完结（全部阶段走完） 或 订单作废
```

---

## 二、核心流程（ASCII 流程图）

```
          ┌──────── 入口 3 种 ────────┐
          │ ① 手工新建 ② 来源合同     │
          │ ③ （扩展）来源商机/报价   │
          └─────────────┬─────────────┘
                        ▼
               OrderService.add（写 Order + Field）
                关联 customerId + 可选 contractId
                        │
                        ▼
          OrderStageService.init → 初始阶段（stageConfig.sort 最小）
                        │
                        ▼
          ┌───── 阶段推进循环 ──────┐
          │ OrderStageController   │
          │ 手动点下一阶段           │
          │ → 阶段配置校验条件       │
          │ → 更新 stageId          │
          │ → 打 OrderSnapshot      │
          └───────────┬─────────────┘
                      │ 阶段推进到最后一个阶段（"已完成"）
                      ▼
              【订单自动完结】
              或管理员手动完结
                      │
                      ▼
              订单作废（异常）
              → 作废原因 + 快照
              → 不允许写操作
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 订单创建 | 订单页"新建订单"或合同详情"生成订单" | 字段校验 → customerId 必填 → contractId 选填 → 写 Order + OrderField + 初始阶段 + 操作日志 + 首份快照 | OrderService.add |
| 阶段推进 | 点击"下阶段"按钮 | 校验前置条件（配置化）→ 更新 order.stageId → 操作日志 + OrderSnapshot | OrderStageService.advanceStage(StageSortRequest) |
| 阶段排序/配置 | 管理员管理→订单阶段设置 | OrderStageConfig CRUD：新增/编辑/删除/排序 sort；保存后缓存失效 | OrderStageController（admin） |
| 订单导出/导入 | Excel 导入导出 | OrderExportService 分页+流式；大导出异步；导入按 system/Import（ImportType.ORDER） | OrderExportService |
| 订单视图筛选 | 订单列表"视图"Tab | 委托 UserViewService（system）：保存筛选条件+公开视图 | OrderUserViewController + UserViewService |
| 订单作废 / 完结 | 订单页操作按钮 | 作废：voidReason + 状态作废 + Snapshot；完结：状态 completed + Snapshot | OrderService.void + complete |

---

## 三、核心业务规则

| 规则 ID | 规则描述 | 例外 |
|---|---|---|
| R1 | **客户必选**：if (customerId 为空 OR 客户无效) then 拒绝创建订单 | 草稿订单可暂不选客户（前端弹提示） |
| R2 | **阶段不可逆（默认）**：if (目标阶段 sort < 当前阶段 sort) then 默认拒绝（开关"允许阶段回退"打开例外） | 管理员强制回退（写操作日志+风险原因） |
| R3 | **来源合同一致性**：if (订单.contractId != null AND 订单.customerId != 合同.customerId) then 拒绝创建/修改 | — |
| R4 | **快照关键节点**：创建/阶段推进/作废/完结 必须打 OrderSnapshot；字段变更 diff 走操作日志 | — |
| R5 | **作废后锁定**：if (订单.status == 作废) then 禁止一切 UPDATE/DELETE（只能查看+导出） | 管理员"撤销作废"权限节点可以恢复 |

---

## 四、状态/枚举（定位入口）

| 语义 | 类名入口 | 代码路径 |
|---|---|---|
| 订单阶段 | OrderStageConfig + sort 排序；Order.stageId 外键 | crm.order.domain.OrderStageConfig |
| 订单状态（草稿/进行中/已完成/作废） | Order.status 字段 | crm.order.domain.Order |
| 表单 Key | FormKey.ORDER | common.constants.FormKey |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **customer** | order ↔ customer | 订单必须关联 customer.customerId |
| **contract** | order ← contract（来源） | Order.contractId = 来源合同；合同完结时可触发"自动创建订单"（若配置开） |
| **product** | order → product（订单明细，若系统有订单明细） | Order 字段扩展中 product 引用（核对订单明细字段） |
| **system** | order → system | 模块字段（OrderField/Blob）、操作日志（OrderLogService）、导出(ExportTask)、导入、视图(UserView)、阶段高级配置(StageAdvancedConfig) |

---

## 六、常见边界场景

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **阶段配置删除老订单卡阶段** | 同商机模式：保持原 stageId，前端标"废弃阶段"引导切换到新配置 | OrderStageService |
| **大订单导出** | 分页+流式+异步；导出字段按 ModuleField 权限裁剪 | OrderExportService + tech/excel-io.md |
| **合同已作废但订单还在进行** | 订单不自动作废（由业务判断）；合同作废时给订单负责人发通知；订单详情展示"关联合同已作废"警告 | Notice + OrderController.get |
| **重复创建订单（同一合同多次创建）** | 允许同一合同创建多份订单（分批次执行）；可通过"合同订单数"字段或关联查询统计 | OrderService.countByContractId |
