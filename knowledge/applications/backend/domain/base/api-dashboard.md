---
type: domain-base
sub-type: api-index
module: dashboard
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/dashboard/controller 目录（或单独 dashboard 应用，核对代码）
owner: cordys-crm
verify-required: true
verify-note: 仪表盘 JSON 结构、小部件类型、DataEase embed 参数 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/dashboard.md
---

# 仪表盘 模块 API 索引（api-dashboard）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **DashboardController** | 仪表盘 CRUD、仪表盘共享、收藏、默认仪表盘设置、仪表盘排序 | DashboardService | `DASHBOARD:ADD/EDIT/DELETE/VIEW/SHARE/SET_DEFAULT` |
| **DashboardWidgetController** | 仪表盘内的小部件 CRUD、小部件尺寸/位置保存、小部件数据查询接口（按 dataSource） | DashboardWidgetService + 各数据查询 Provider（线索/客户/商机…/DataEase ） | `DASHBOARD:WIDGET_*` |
| **DashboardCollectionController**（如有） | 仪表盘收藏夹/收藏/取消收藏/我的收藏列表 | DashboardCollectionService | `DASHBOARD:COLLECTION_*` |
| **DashboardCatalogController**（如有） | 仪表盘分类/目录 CRUD、排序 | DashboardCatalogService | `DASHBOARD:CATALOG_MANAGE` |

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 仪表盘列表/新建/编辑/删除/共享/设为默认 | DashboardController |
| 小部件（图表/卡片/指标卡）增删改、保存位置/尺寸 | DashboardWidgetController |
| 收藏/我的收藏夹 | DashboardCollectionController（核对） |
| 仪表盘分类 | DashboardCatalogController（核对） |

路径前缀（核对 @RequestMapping）：
```
/api/dashboard/*           → 仪表盘
/api/dashboard-widget/*    → 小部件
/api/dashboard-collection/* → 收藏
/api/dashboard-catalog/*   → 分类
```
