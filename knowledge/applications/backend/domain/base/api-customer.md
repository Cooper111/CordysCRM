---
type: domain-base
sub-type: api-index
module: customer
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/customer/controller 目录
owner: cordys-crm
verify-required: true
verify-note: 实际接口签名/路径需回代码核对（易变项 R4）
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/customer.md
  - knowledge/applications/backend/domain/base/domain-customer.md
  - knowledge/applications/backend/domain/base/repository-customer.md
---

# 客户模块 API 索引（api-customer）

> 定位入口文档，非接口文档。实际签名以 Controller 源码为准。

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 代码路径前缀 | 对应 Service | 权限节点前缀（核对 permission.json） |
|---|---|---|---|---|
| **CustomerController** | 客户主档 CRUD、线索转客户、客户反转化回线索、客户合并、客户移交、批量编辑、导入导出入口、统计图表 | cn.cordys.crm.customer.controller.CustomerController | CustomerService + CustomerExportService | `CUSTOMER:ADD/EDIT/DELETE/VIEW/TRANSFORM/MERGE/TRANSFER/BATCH_EDIT/EXPORT` |
| **CustomerContactController** | 客户联系人 CRUD、联系人查重、联系人批量编辑、导入导出 | CustomerContactController | CustomerContactService | `CUSTOMER:CONTACT_ADD/EDIT/DELETE/VIEW` |
| **CustomerCollaborationController** | 客户协作人：增删改查、变更协作类型（只读/编辑） | CustomerCollaborationController | CustomerCollaborationService | `CUSTOMER:COLLABORATION_MANAGE` |
| **CustomerPoolController** | 客户公海管理（管理员）：客户池 CRUD、字段隐藏配置、池分配规则/回收规则、批量投放客户、回池 | CustomerPoolController | CustomerPoolService + PoolCustomerService | `CUSTOMER:POOL_MANAGE / POOL_PICK / POOL_VIEW` |
| **CustomerFollowPlanController** | 客户→跟进计划（代理 follow，resourceType=CUSTOMER） | CustomerFollowPlanController | FollowUpPlanService（follow） | `CUSTOMER:FOLLOW_PLAN_*` |
| **CustomerFollowRecordController** | 客户→跟进记录（代理 follow，resourceType=CUSTOMER） | CustomerFollowRecordController | FollowUpRecordService（follow） | `CUSTOMER:FOLLOW_RECORD_*` |
| **CustomerUserViewController** | 客户列表筛选视图（保存筛选条件） | CustomerUserViewController | UserViewService（system） | `CUSTOMER:VIEW + USER_VIEW:*` |
| （如有 CustomerCapacityController） | 客户容量查询/管理员调整 | CustomerCapacityController（核对是否有） | CustomerCapacityService | `CUSTOMER:CAPACITY_VIEW / ADJUST` |

---

## 二、定位技巧

| 需求关键词 | Controller |
|---|---|
| 客户列表/详情/新建/编辑/合并/移交/反转化 | CustomerController |
| 客户联系人（新增/修改/查重） | CustomerContactController |
| 客户协作人（谁能看能改） | CustomerCollaborationController |
| 客户公海（池管理/领取/退回） | CustomerPoolController |
| 客户 Tab：跟进计划 / 跟进记录 | CustomerFollowPlanController / CustomerFollowRecordController |
| 客户列表 保存视图条件 | CustomerUserViewController |

路径前缀（常见模式，核对代码 @RequestMapping）：
```
/api/customer/*              → CustomerController
/api/customer-contact/*      → CustomerContactController
/api/customer-collaboration/* → CustomerCollaborationController
/api/customer-pool/*         → CustomerPoolController
/api/customer-follow-plan/*  → FollowPlan（客户）
/api/customer-follow-record/* → FollowRecord（客户）
/api/customer-view/*         → 客户筛选视图
```
