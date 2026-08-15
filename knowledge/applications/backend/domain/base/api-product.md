---
type: domain-base
sub-type: api-index
module: product
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/product/controller 目录
owner: cordys-crm
verify-required: true
verify-note: 产品字段扩展/版本/价格策略 接口签名易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/product.md
  - knowledge/applications/backend/domain/base/domain-product.md
---

# 产品模块 API 索引（api-product）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀（核对 permission.json） |
|---|---|---|---|
| **ProductItemController** | 产品主档 CRUD、产品字段（字段扩展/字段Blob）、产品导入导出、产品启用/停用、（若没有独立 VersionController）产品多版本操作 | ProductItemService + ProductFieldService + ProductExportService | `PRODUCT:ADD/EDIT/DELETE/VIEW/EXPORT/VERSION_MANAGE` |
| **ProductCategoryController**（独立类则存在） | 产品分类树 CRUD、分类排序、分类启用/停用、下级递归删除校验 | ProductCategoryService | `CATEGORY:ADD/EDIT/DELETE/VIEW` |
| **ProductPriceBookController** | 价格簿 CRUD、价格簿明细 CRUD、价格簿批量导入/导出、价格簿启用/停用 | ProductPriceBookService + ProductPriceBookItemService | `PRICEBOOK:ADD/EDIT/DELETE/VIEW/IMPORTEXCEL/EXPORTEXCEL` |

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 产品列表/新建/编辑/删除/字段配置/导入导出 | ProductItemController |
| 产品分类树管理 | ProductCategoryController |
| 价格簿 CRUD / 价格明细 | ProductPriceBookController |
| 产品多版本复制/切换 | 核对 ProductVersionController 独立类？若否则在 ProductItemController 内 |

路径前缀（核对 @RequestMapping）：
```
/api/product-item/*          → 产品主档
/api/product-category/*      → 产品分类
/api/product-price-book/*    → 价格簿
/api/product-price-book-item/* → 价格簿明细（如独立）
```
