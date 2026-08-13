---
type: domain-product
app: backend
module: dashboard
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/dashboard 下 Dashboard / DashboardModule / DashboardCollection（收藏夹）+ 对应 Service / Mapper / Controller
owner: cordys-crm
verify-required: true
verify-note: 图表类型枚举/模块配置 JSON 结构/拖拽排序的位置算法 需回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#仪表盘
  - knowledge/applications/backend/domain/base/api-dashboard.md
  - knowledge/applications/backend/tech/mybatis-plus.md（BaseMapper 用法）
---

# 仪表盘模块 · 产品能力

---

## 一、能力概述

提供 CRM 首页仪表盘 / 工作台自定义看板能力：多张仪表盘（Dashboard）+ 每个仪表盘多个模块（DashboardModule，支持 柱状/折线/饼/漏斗/指标卡/列表卡 等图表类型）+ 拖拽排序 + 收藏夹（DashboardCollection：用户可以收藏常用仪表盘快速切换）。

图表数据来源：由前端组件按图表类型调各业务模块的统计 API（线索/客户/商机/合同/订单/跟进统计），本模块负责"仪表盘配置"（谁能用哪些仪表盘、每个仪表盘的模块布局与配置、排序）。

---

## 二、核心流程（ASCII 流程图）

```
管理员：DashboardService.create
    ├ 新建仪表盘 + 设为"公开"或"指定角色可见"
    ├ DashboardModuleService：添加模块（柱状/折线/饼/漏斗/指标卡/列表...）
    │    ├ 每个模块：图表配置 JSON（数据源、维度、指标、筛选）
    │    └ sort 排序（模块在仪表盘中的位置：sort 越小越靠前）
    └ DashboardSortService：拖拽保存 → 更新模块 sort

用户使用：
    ├ Dashboard 列表：公开 + 我被分配的 + 我收藏的（DashboardCollection）
    ├ 点击进入仪表盘：前端按 DashboardModule 顺序渲染图表组件
    │    └ 每个图表组件 → 调对应业务的统计 API（如 ClueService.stat、OpportunityService.amountStat）
    ├ 收藏/取消收藏 Dashboard → 写 DashboardCollection（user_id + dashboard_id）
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 仪表盘 CRUD | 管理后台仪表盘 | Dashboard + DataScope 过滤（公开/指定角色可见）；保存后缓存失效 | DashboardService.add / update / delete |
| 仪表盘模块 CRUD | 仪表盘编辑→模块编辑 | DashboardModule：每个模块配置 JSON（图表类型、数据源接口、筛选条件、排序字段）→ 写 DashboardModule 表 | DashboardModuleService |
| 拖拽排序 | 前端拖拽模块后调用保存接口 | DashboardSortService：模块移动后重新计算 sort（经典"链表重排"算法或"插空 + 重置同层所有 sort"） | DashboardSortService.saveSort（MoveNodeSortDTO / DropNode 等 DTO） |
| 收藏/取消收藏 | 用户在仪表盘列表点 ★ | DashboardCollection：user_id + dashboard_id 唯一键 → 新增或删除；避免重复 | DashboardService.collect / uncollect + DashboardCollectionMapper |
| 仪表盘树展示（如有） | 仪表盘分类（DashboardTreeNode） | 支持分组（分类）：DashboardTreeNode DTO（若实现）；未分组则扁平列表 | DashboardService.tree / listTreeNode |

---

## 三、核心业务规则

| 规则 ID | 规则描述 | 例外 |
|---|---|---|
| R1 | **默认仪表盘**：每个组织至少 1 张"系统默认仪表盘"（由超管设置 isDefault=true）；新用户首次进首页自动打开它 | — |
| R2 | **数据来源信任前端配置**：后端 DashboardModule 只负责"保存配置 JSON + 按权限返回"，不做图表数据计算；具体图表数据由各业务 Service 负责统计（受 CsPermission 过滤） | — |
| R3 | **删除仪表盘：私有仪表盘仅所有者可删；公开仪表盘需管理员** | 超管可删除任意 |
| R4 | **公开仪表盘被删用户无法打开**：用户之前收藏的仪表盘被删除后 → DashboardCollection 自动失效（查询时 join Dashboard 过滤 status!=删除） | — |

---

## 四、状态/枚举（定位入口）

| 语义 | 枚举/类名入口 | 代码路径 |
|---|---|---|
| 仪表盘类型（系统默认/自定义/私有） | Dashboard.type 字段 | crm.dashboard.domain.Dashboard |
| 图表类型（柱状/折线/饼/漏斗/指标卡/列表） | DashboardModule.chartType 字段 + 枚举或字典 | crm.dashboard.domain.DashboardModule |
| 模块排序 DTO | DashboardTreeNode / MoveNodeSortDTO / DropNode | crm.dashboard.dto.* |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **各业务模块（统计 API）** | dashboard ← 业务（图表组件） | DashboardModule 数据源接口配置 → 前端组件调各业务统计接口：clue/customer/opportunity/contract/order/follow/home.statistic.* |
| **system（角色权限/操作日志）** | dashboard → system | 操作日志 DashboardLogDTO；仪表盘按角色分配可见范围（用户登录时返回可见列表） |

---

## 六、常见边界场景

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **同一仪表拖拽模块过多（>50）导致性能差** | 默认分页返回模块（DashboardModule 不分页，实际业务很少 >30）；50+ 时前端按屏渲染（懒加载） | DashboardModuleService.listByDashboardId |
| **拖拽排序 sort 冲突并发** | DashboardSortService.saveSort 加分布式锁（Redis key：dashboard_sort_lock{dashboardId}），每次重排所有模块 sort，避免冲突 | DashboardSortService 加锁逻辑 |
| **统计图表接口超时（大数据量）** | 图表组件统一 15s 超时 + 前端 fallback 展示"数据加载中…请稍后重试"；超时可离线同步到 DataEase 做 BI 报表（超大统计建议直接用 DataEase 嵌入式） | 前端组件统一超时配置 |
