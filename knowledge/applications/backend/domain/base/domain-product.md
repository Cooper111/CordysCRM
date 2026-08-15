---
type: domain-base
sub-type: domain-index
module: product
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/product/ 目录
owner: cordys-crm
verify-required: true
verify-note: 产品字段扩展多/版本号/价格字段 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/product.md
  - knowledge/applications/backend/domain/base/api-product.md
---

# 产品模块 领域模型索引（domain-product）

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **ProductItem** | 产品主档：产品编码、产品名称、分类 id、规格、型号、单位、启用状态、多版本号（若内嵌）、是否下架 | crm_product_item | ProductItemExtMapper | Y（ProductField + ProductFieldBlob，产品字段扩展常用） |
| **ProductField** | 产品扩展字段行 | crm_product_field | ProductFieldExtMapper | N |
| **ProductFieldBlob** | 产品扩展 BLOB | crm_product_field_blob | ProductFieldBlobExtMapper | N |
| **ProductCategory** | 产品分类树：父级 id、分类名称、排序、是否启用 | crm_product_category | ProductCategoryExtMapper | N |
| **ProductVersion**（若独立类） | 产品版本：产品 id、版本号、版本生效时间、版本说明、是否当前版本、复制来源版本 id | crm_product_version（核对表名） | ProductVersionExtMapper | N |
| **ProductPriceBook** | 价格簿：价格簿名称、币种、生效日期/失效日期、启用状态、是否基准价格簿 | crm_product_price_book | ProductPriceBookExtMapper | N |
| **ProductPriceBookItem** | 价格簿行项：价格簿 id、产品 id、（若产品多版本：版本 id）、单价、阶梯价 JSON（按数量阶梯）、折扣下限 | crm_product_price_book_item | ProductPriceBookItemExtMapper | N |
| **ProductReference** | 产品引用计数器（统计被多少条报价/订单产品引用，防止误删）：productId、type（QUOTATION_ITEM/ORDER_PRODUCT）、count、lastRefTime | crm_product_reference（如有） | ProductReferenceExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.product.dto.*`
- export：`cn.cordys.crm.product.dto.export.*`

---

## 二、定位技巧

| 关键词 | 类 |
|---|---|
| 产品列表/详情/产品编号/名称/规格/启用/下架 | ProductItem |
| 产品自定义字段 / 列表多了个字段 | ProductField + ProductFieldBlob |
| 产品分类 / 分类树 / 分类排序 | ProductCategory |
| 产品多版本 / 版本切换 / 复制新版本 | ProductVersion（若独立，否则嵌套在 ProductItem） |
| 价格簿 / 不同客户群不同价格 | ProductPriceBook |
| 某产品 在某价格簿 中的 价格 / 阶梯价 | ProductPriceBookItem |
| 产品被报价/订单 引用 了多少次 / 删不掉 提示"被引用" | ProductReference（或各业务 Service 自己 count，核对实际实现） |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **ProductStatusEnum** | 产品状态（草稿/已上架/已下架/停用…，常字典表） |
| **PriceBookTypeEnum** | 价格簿类型（基准价/客户价/项目价…，核对） |
