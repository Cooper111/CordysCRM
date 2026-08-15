---
type: domain-base
sub-type: api-index
module: home
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/home/controller 目录（或 HomeController 单文件）
owner: cordys-crm
verify-required: true
verify-note: 首页统计指标口径 易变（如"新增线索数"是否含已转化），核对 Service 实现
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/home.md
---

# 首页统计 模块 API 索引（api-home）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **HomeController** | 首页"统计卡片""待办事项""我的待跟进""我的提醒""快过期数据""数据概览"等聚合查询接口 | HomeStatisticsService（内部聚合 ClueService/CustomerService/ApprovalService/FollowReminderTaskService 等） | `HOME:VIEW`（一般登录即可，核对 permission.json） |

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 首页 / 工作台 / 数据概览 / 我的待办 / 我的提醒 聚合接口 | HomeController |
| 某模块具体统计的口径问题 | 去 HomeStatisticsService 看具体调用链，最终走对应子模块 Service（如 ClueService.pageCount 或 customerService.statXxx） |

路径前缀（核对 @RequestMapping）：
```
/api/home/* → HomeController（通常 /api/home/statistics, /api/home/todo, /api/home/reminder 等子路径）
```
