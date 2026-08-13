---
type: domain-base
sub-type: api-index
module: clue
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/clue/controller 目录（9 个 Controller）
owner: cordys-crm
verify-required: true
verify-note: 具体 REST 路径、方法签名、请求参数、返回字段 均为易变项，必须回到代码实际核对
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/clue.md
  - knowledge/applications/backend/domain/base/domain-clue.md
  - knowledge/applications/backend/domain/base/repository-clue.md
---

# 线索模块 API 索引（api-clue）

> 本文档仅提供**快速定位入口**，按 Controller 类名 + 代码路径即可找到对应源码。
> **禁止**把本文件当"接口文档"使用：接口签名、HTTP 方法、参数顺序、返回结构 均为易变项，直接读 Controller 源码。

---

## 一、Controller 类定位（9 个）

| Controller 类名 | 职责一句话 | 代码路径 | 对应 Service | 权限节点前缀（**值核对 permission.json**） |
|---|---|---|---|---|
| **ClueController** | 线索主档 CRUD：单条/批量增改删查、线索转客户、线索分配、线索批量编辑、图表统计、导入入口 | `backend/crm/src/main/java/cn/cordys/crm/clue/controller/ClueController.java` | ClueService | `CLUE:ADD / EDIT / DELETE / EXPORT / VIEW / TRANSFORM / TRANSFER / BATCH_EDIT` |
| **PoolClueController** | 公海（池外）线索：销售领取线索、池内线索列表/详情、领取前校验 | `crm.clue.controller.PoolClueController.java` | PoolClueService | `CLUE:POOL_PICK / POOL_VIEW` |
| **CluePoolController** | 线索池管理（管理员）：线索池 CRUD、池字段隐藏配置、池分配规则、池回收规则、池内投放/批量回池 | `crm.clue.controller.CluePoolController.java` | CluePoolService | `CLUE:POOL_MANAGE（POOL_ADD/POOL_EDIT/POOL_DELETE/POOL_RULE）` |
| **ClueCapacityController** | 线索容量：查看某用户当前线索容量使用情况、管理员手动调整容量配额（临时） | `crm.clue.controller.ClueCapacityController.java` | ClueCapacityService | `CLUE:CAPACITY_VIEW / CAPACITY_ADJUST` |
| **ClueOwnerHistoryController** | 线索负责人变更历史：查询某线索的 owner 历史列表 | `crm.clue.controller.ClueOwnerHistoryController.java` | ClueOwnerHistoryService | `CLUE:OWNER_HISTORY` |
| **PoolClueUserViewController** | 公海线索的筛选视图：池内线索保存用户筛选条件 | `crm.clue.controller.PoolClueUserViewController.java` | UserViewService（system） | `CLUE:POOL_VIEW + USER_VIEW:*` |
| **ClueUserViewController** | 线索列表筛选视图：线索列表保存用户筛选条件（我的视图/公开视图） | `crm.clue.controller.ClueUserViewController.java` | UserViewService（system） | `CLUE:VIEW + USER_VIEW:*` |
| **ClueFollowPlanController** | 线索→跟进计划：为某线索创建/查询跟进计划（代理 follow.FollowUpPlanService，传 resourceType=CLUE） | `crm.clue.controller.ClueFollowPlanController.java` | FollowUpPlanService（follow） | `CLUE:FOLLOW_PLAN_ADD / EDIT / DELETE` |
| **ClueFollowRecordController** | 线索→跟进记录：为某线索写跟进记录（代理 follow.FollowUpRecordService，传 resourceType=CLUE） | `crm.clue.controller.ClueFollowRecordController.java` | FollowUpRecordService（follow） | `CLUE:FOLLOW_RECORD_ADD / EDIT / DELETE` |

---

## 二、定位技巧

### 2.1 按关键词找接口

| 需求关键词 | 找哪个 Controller |
|---|---|
| 线索列表 / 详情 / 新建 / 编辑 / 删除 | ClueController |
| 销售从公海领取线索 | PoolClueController |
| 管理员维护线索池 / 配置规则 / 投放线索 | CluePoolController |
| 线索容量 / 容量不够 / 调整容量 | ClueCapacityController |
| 谁把线索给了谁 / 负责人变更历史 | ClueOwnerHistoryController |
| 线索/公海 保存筛选条件视图 | ClueUserViewController / PoolClueUserViewController |
| 线索 跟进计划 Tab / 跟进记录 Tab | ClueFollowPlanController / ClueFollowRecordController |

### 2.2 按 REST 路径前缀找（常见模式，实际值核对代码 @RequestMapping）

```
/api/clue/*              → ClueController
/api/pool-clue/*         → PoolClueController
/api/clue-pool/*         → CluePoolController
/api/clue-capacity/*     → ClueCapacityController
/api/clue-owner-history/* → ClueOwnerHistoryController
/api/pool-clue-view/*    → PoolClueUserViewController
/api/clue-view/*         → ClueUserViewController
/api/clue-follow-plan/*  → ClueFollowPlanController
/api/clue-follow-record/* → ClueFollowRecordController
```

**说明**：路径前缀是"常见约定"，不是最终事实。实际路径必须核对 Controller 类上的 `@RequestMapping` 注解（易变项 R4）。

---

## 三、相关联的 service / mapper / domain 文件

详见：
- [product/clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/clue.md) 六节"协作关系+边界场景"
- [domain-clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-clue.md) Domain 类清单
- [repository-clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-clue.md) Mapper/表清单
