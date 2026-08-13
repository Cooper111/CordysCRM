---
type: domain-base
sub-type: repository-index
module: product
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/product/mapper/
owner: cordys-crm
verify-required: true
verify-note: 表名/字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-product.md
---

# 产品模块 存储索引（repository-product）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀 |
|---|---|---|
| **ExtProductItemMapper** | 产品主档 CRUD/分页 JOIN 扩展字段 | crm_product_item |
| **ExtProductFieldMapper / Blob** | 产品扩展字段 | crm_product_field / _blob |
| **ExtProductCategoryMapper** | 产品分类树 | crm_product_category |
| **ExtProductVersionMapper**（若独立类） | 产品多版本 | crm_product_version |
| **ExtProductPriceBookMapper** | 价格簿主档 | crm_product_price_book（或 crm_price_book，核对） |
| **ExtProductPriceBookItemMapper** | 价格簿行项 | crm_product_price_book_item |
| **ExtProductReferenceMapper**（如有） | 产品引用计数（被报价/订单引用次数） | crm_product_reference |

XML 路径：`resources/mapper/product/Ext{Xxx}Mapper.xml`

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_product_tables.sql`

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 产品分类递归删除 / 子分类列表 SQL | ExtProductCategoryMapper.xml selectChildrenByParentId |
| 价格簿 阶梯价 JSON 字段如何存 | 看 ProductPriceBookItemMapper 插入/更新逻辑 + 表字段 price_ladder_json |
| 删产品时 报错"被订单引用" | ExtProductReferenceMapper.countByProductId 或各业务 Mapper 独立 count，核对实现 |
