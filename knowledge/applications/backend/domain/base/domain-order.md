---
type: domain-base
sub-type: domain-index
module: order
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/order/ 目录
owner: cordys-crm
verify-required: true
verify-note: 订单阶段/发货状态/产品明细字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/order.md
  - knowledge/applications/backend/domain/base/api-order.md
---

# 订单模块 领域模型索引（domain-order）

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **Order** | 订单主档：订单编号、关联合同/商机、客户、下单日期、交付日期、订单金额（价税合计）、阶段(draft/confirmed/shipped/received/completed/returned/void)、发货状态/收货状态、负责人 | crm_order | OrderExtMapper | Y（OrderField + OrderFieldBlob） |
| **OrderField** | 订单扩展字段行 | crm_order_field | OrderFieldExtMapper | N |
| **OrderFieldBlob** | 订单扩展 BLOB | crm_order_field_blob | OrderFieldBlobExtMapper | N |
| **OrderProduct** | 订单产品行项：订单 id、产品 id、产品快照 JSON、数量、单价、折扣率、税额、小计、交付状态 | crm_order_product | OrderProductExtMapper | N |
| **OrderStage** | 订单阶段配置（如独立） | crm_order_stage（核对） | OrderStageExtMapper | N |
| **OrderStageHistory** | 订单阶段变更历史 | crm_order_stage_history | OrderStageHistoryExtMapper | N |
| **OrderDispatchNote** | 发货单（一订单可多次发货）：发货日期、物流单号、物流公司、收货地址快照、发货人、备注 | crm_order_dispatch_note（核对表名） | OrderDispatchNoteExtMapper | N |
| **OrderDispatchItem** | 发货单-产品明细（按订单产品拆分发货数量） | crm_order_dispatch_item（核对） | OrderDispatchItemExtMapper | N |
| **OrderReturnReceipt** | 回签单/收货确认单：关联发货单、实际签收日期、签收人、签收附件、是否无异议 | crm_order_return_receipt（核对） | OrderReturnReceiptExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.order.dto.*`
- export：`cn.cordys.crm.order.dto.export.*`

---

## 二、定位技巧

| 关键词 | 类 |
|---|---|
| 订单本身 / 编号 / 金额 / 阶段 | Order |
| 订单自定义字段 | OrderField + Blob |
| 订单里有哪些产品/数量/折扣/税额 | OrderProduct |
| 订单阶段 / 阶段变更轨迹 | OrderStage + OrderStageHistory |
| 发货 / 物流 / 发了哪几批 / 发货单 | OrderDispatchNote + OrderDispatchItem |
| 回签单 / 客户有没有签收 / 收货确认 | OrderReturnReceipt |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **OrderStatusEnum / OrderStageEnum** | 订单状态/阶段 |
| **ShipStatusEnum** | 发货状态（未发/部分发/已发完） |
| **ReceiveStatusEnum** | 收货状态（未收/部分收/已收完） |
