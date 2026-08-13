---
type: domain-product
app: backend
module: product
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/product 下 Product / Price / ProductField / PriceField / ProductService / ProductPriceService / ProductExportService + ProductUtils
owner: cordys-crm
verify-required: true
verify-note: 价格版本生效逻辑/报价引用计数/字段类型配置需回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#产品/价格表
  - knowledge/main/business-rules.md#权限模型
  - knowledge/applications/backend/domain/base/api-product.md
  - knowledge/applications/backend/domain/base/domain-product.md
  - knowledge/applications/backend/domain/base/repository-product.md
---

# 产品模块 · 产品能力

---

## 一、能力概述

产品模块管理公司销售的**产品主档（SKU）**和**价格体系（多版本价格表）**，是商机报价、合同、订单的上游基础数据。包含：产品 CRUD + 自定义字段（ProductField）、价格版本（Price）与价格明细（PriceField）、产品导入导出、视图筛选。

主干流程：
```
产品新建（产品字段配置化） → 维护基础信息（分类/规格/单位）
       ↓
  价格版本（多个版本并存：试价期/正式/促销）
       ↓
  下游商机报价 / 合同/订单 → 引用 Product.productId + Price.priceId（取生效版本价）
```

---

## 二、核心流程（ASCII 流程图）

```
        ┌──────── 产品生命周期 ─────────┐
        │                              │
   产品 + 字段 新建 ──→ 启用 ──→ 报价/合同/订单引用 ──→ 停用
        │       └ 编辑修改（产品名称/规格）        ↑
        ▼                                          │
  价格版本（可多版本）                              │  停用前校验：
   ├ 生效时间 from/to                               │  无未完结报价引用
   ├ 价格明细（PriceField）                         │  无未完结合同/订单
   └ 版本切换 → 改默认生效版本                      │
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 产品新建/编辑 | 产品管理页 CRUD | 字段校验（必填/唯一）→ ProductService 写 Product + ProductField + ProductFieldBlob → 操作日志（ProductLogService） | ProductService.add/update |
| 产品字段配置 | 管理中心→模块字段→产品字段 | 由 system/ModuleField 统一管理；产品字段 = BusinessModuleField.PRODUCT → 字段类型/必填/唯一性配置 → 前端动态渲染 | system.service.ModuleFieldService + ProductFieldService |
| 价格版本创建 | 产品详情→价格 Tab→新建版本 | 价格版本：有效期 fromDate / toDate + 状态（草稿/生效/过期）+ 价格明细 PriceField → 默认生效版本只能一个（切换时写） | ProductPriceService.add + activateVersion |
| 报价/合同/订单引用产品 | 商机报价明细→选产品 | 取产品最新"生效价格版本"（按当前日期在有效期内的默认版本） → 带回 price/unit 等作为默认报价价（可手动改） | ProductService.queryForQuotation + ProductPriceService.getCurrentEffectivePrice |
| 产品停用 | 管理员停用产品 | 停用前：检查该产品是否被"进行中报价/未完结合同/订单"引用 → 有引用则拒绝停用；无则标记停用 | ProductService.disable（具体名核对代码） |
| 导入/导出 | Excel 导入导出 | ProductExportService：分页+流式，大导出异步；导入按 system/Import | ProductExportService + ProductUtils/ProductPriceUtils |

---

## 三、核心业务规则

| 规则 ID | 规则描述 | 例外 |
|---|---|---|
| R1 | **产品编码唯一**：if (同组织内 productCode 重复) then 拒绝创建/编辑 | 合并产品时管理员强制去重 |
| R2 | **默认生效价格版本唯一**：同一产品同一时间点 → 只能有 1 个默认版本 = true | 超管可临时指定两个版本（灰度/切换期） |
| R3 | **版本价格不允许编辑（合规）**：if (Price 状态=生效) then 价格明细只读；修改方式："复制版本→改价→生效新→停用旧" | — |
| R4 | **停用产品校验引用**：if (被 OPPORTUNITY_QUOTATION_DETAIL / CONTRACT_DETAIL / ORDER_DETAIL 引用且状态=进行中) 则拒绝停用 | 管理员强制停用 + 提示"相关报价/合同/订单需同步处理" |
| R5 | **报价默认价格来源**：默认价 = 当前产品默认生效版本的价格；如果当前日期没有生效版本 → 报"该产品没有生效价格"的错，提示先建价格版本 | 管理员可以直接填 0 手动改价格 |

---

## 四、状态/枚举（定位入口）

| 语义 | 类名入口 | 代码路径 |
|---|---|---|
| 产品状态（启用/停用） | Product.status 字段 | crm.product.domain.Product |
| 价格版本状态（草稿/生效/过期） | Price.status + fromDate/toDate 共同决定 | crm.product.domain.Price |
| 产品分类（字典） | DictModule.PRODUCT_CATEGORY | system.constants.DictModule |
| 表单 Key | FormKey.PRODUCT / PRODUCT_PRICE | common.constants.FormKey |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **opportunity（报价）** | product ← opportunity | 报价明细取 productId + priceId：OpportunityQuotation→ProductService/ProductPriceService 查询当前价 |
| **contract / order** | product ← contract/order | 合同/订单明细（如有）引用产品 |
| **system** | product → system | ProductLogService + PriceLogService（操作日志）、ModuleField（字段）、导入导出 |
| **dashboard** | dashboard → product | 销售额报表按产品维度聚合（由订单/合同金额 join productId） |

---

## 六、常见边界场景

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **价格版本每天/每周切换（频繁调价）** | 生效时间段按自然周期（toDate+1 自动失效）；提前 3 天给管理员发通知"价格版即将到期" | ProductPriceService 过期监听器 |
| **批量导入产品（3000+）** | 异步 Import + 批处理提交（每 500 条 flush）；失败行汇总下载 | ProductExportService + System.Import |
| **产品改名后历史报价跟着变（不期望）** | 报价快照/合同快照：报价明细保存时"同时冗余产品名称/规格 JSON" → 展示用冗余值，改名不影响历史 | OpportunityQuotationSnapshot / ContractSnapshot 冗余字段 |
| **删除产品（合规不留痕风险）** | 默认不提供 DELETE 接口，只有停用；管理员 DELETE 前强制 check 引用计数 = 0 | ProductService.delete（如有） |
