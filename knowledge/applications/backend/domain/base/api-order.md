---
type: domain-base
sub-type: api-index
module: order
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/order/controller 目录
owner: cordys-crm
verify-required: true
verify-note: 订单发货/收货/退货 流程接口参数易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/order.md
  - knowledge/applications/backend/domain/base/domain-order.md
---

# 订单模块 API 索引（api-order）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **OrderController** | 订单主档 CRUD、订单阶段推进、发货、收货确认、订单作废、批量编辑、导入导出入口、统计图表 | OrderService + OrderStageService + OrderExportService | `ORDER:ADD/EDIT/DELETE/VIEW/VOID/ itemSHIP/RECEIVE/BATCH_EDIT/EXPORT` |
| **OrderProductController** | 订单产品明细 CRUD中原：订单产品数量、折扣、税额、产品快照字段配置（与 Form/的 ModuleField 计数） | OrderProductService | `ORDER:PRODUCT_ADD/ /EDIT_B/DELETE` |
| **OrderReturnReceipt/OrderDispatchNoteController** | 发货、发货单、回签单，收货流程 接口拆分情况核对代码 | Order Service（实现） | `ORDER:SHIP / ORDER:RECEIVE` |

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 订单本身 CRUD / 阶段推进 / 作废 / 导入导出 | OrderController |
| 订单产品明细 | OrderProductController（或嵌套在 Order 对象，核对代码） |
| 订单发货/收货相关 | 看是否有独立的 OrderShip/OrderReceive 相关 Controller，核对代码 |

路径前缀（核对 @RequestMapping）：
```
/api/order/*          → 订单主档
/api/order-product/*  → 订单明细
/api/order-ship/* /api/order-receive/*（如有）
```
