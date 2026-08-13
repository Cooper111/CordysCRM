---
type: domain-base
sub-type: repository-index
module: order
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/order/mapper/
owner: cordys-crm
verify-required: true
verify-note: 表名/字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-order.md
---

# 订单模块 存储索引（repository-order）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀 |
|---|---|---|
| **ExtOrderMapper** | 订单主档 CRUD/分页 JOIN 扩展字段 | crm_order |
| **ExtOrderFieldMapper / Blob** | 订单扩展字段 | crm_order_field / _blob |
| **ExtOrderProductMapper** | 订单产品行项 | crm_order_product |
| **ExtOrderStageMapper**（若独立） | 订单阶段配置 | crm_order_stage |
| **ExtOrderStageHistoryMapper** | 订单阶段变更历史 | crm_order_stage_history |
| **ExtOrderDispatchNoteMapper** | 发货单 | crm_order_dispatch_note（表名核对） |
| **ExtOrderDispatchItemMapper** | 发货单-产品明细 | crm_order_dispatch_item |
| **ExtOrderReturnReceiptMapper** | 回签单/收货确认 | crm_order_return_receipt |

XML 路径：`resources/mapper/order/Ext{Xxx}Mapper.xml`

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_order_tables.sql`

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 订单发货进度（发了多少/还剩多少） | ExtOrderDispatchItemMapper.xml 汇总 by orderProductId |
| 订单交付预警（快到交付日没发货） | ExtOrderMapper.xml selectDeliveryWarning |
| 订单产品 引用了产品快照（产品改名后订单保留旧名） | 看 ExtOrderProduct.productSnapshot 字段（JSON）的写入逻辑在 OrderService |
