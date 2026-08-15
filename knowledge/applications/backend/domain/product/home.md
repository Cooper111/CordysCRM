---
type: domain-product
app: backend
module: home
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/home 下 HomeStatisticController / HomeStatisticService / HomeStatisticPeriod + response DTO（线索/客户统计）
owner: cordys-crm
verify-required: true
verify-note: 统计口径（是否删除/是否作废/各状态）需回代码核对 HomeStatisticService 实现
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#销售漏斗
  - knowledge/main/core-process.md#1-销售漏斗总览
  - knowledge/applications/backend/domain/base/api-home.md
---

# 首页统计模块 · 产品能力

---

## 一、能力概述

首页（工作台）的统计卡片与销售漏斗数据接口，提供按时间周期（今日/本周/本月/本季度/自定义）的：
- 线索统计（新增线索、待跟进线索、已转化线索、线索回收）
- 客户统计（新增客户、联系人、公海客户、客户容量占用情况）
- （前端扩展）商机、合同、订单、跟进统计（来自对应业务 Service 统计方法）

本模块**只读无写**，所有统计均来自对应业务模块的 Mapper count 查询，不存中间表。

---

## 二、核心流程（ASCII 流程图）

```
用户打开首页（工作台）
        │
        ▼
   前端并行调用统计接口：
   ├ HomeStatisticController.clue（period=今日/本周/本月）
   │     └ HomeStatisticService → ExtClueMapper.countByPeriod
   │         └ 受 DataScopeService 过滤：按当前用户数据范围统计
   ├ HomeStatisticController.customer（period=...）
   │     └ → ExtCustomerMapper.countByPeriod + 容量占用统计
   ├ （可选）商机/合同/订单统计：调对应 Controller 的 statistic 接口
   └ （可选）待跟进/已逾期：直接调 FollowUpPlanRemindListener 对应查询逻辑
        │
        ▼
   返回 HomeClueStatistic / HomeCustomerStatistic DTO → 前端渲染统计卡片 & 销售漏斗
```

---

## 三、核心业务规则

| 规则 ID | 规则描述 | 例外 |
|---|---|---|
| R1 | **统计受数据权限严格限制**：if (当前用户数据范围=SELF) then 只统计自己 owner/协作人的；DEPARTMENT=本部门；ALL=全组织；CUSTOMER_COLLABORATION 算协作人的 | — |
| R2 | **时间周期对齐自然日/周**：period=今日→今天 00:00:00 到 NOW；本周→本周一 00:00 到 NOW；本月→本月 1 号 00:00 到 NOW；季度/年同理 | 自定义日期区间：前端传 startTime/endTime 覆盖 |
| R3 | **过滤已删除/已作废**：统计一律排除 status=已作废/已删除/回收 (具体过滤规则按代码为准) | 管理员可传参数 `includeVoided=true` 查看（默认 false） |

---

## 四、状态/枚举（定位入口）

| 语义 | 枚举/类名入口 | 代码路径 |
|---|---|---|
| 首页统计周期（今日/本周/本月/本季度/本年/自定义） | `cn.cordys.crm.home.constants.HomeStatisticPeriod` | crm.home.constants.HomeStatisticPeriod.java |
| 统计返回 DTO | `HomeClueStatistic / HomeCustomerStatistic`（response DTO） | crm.home.dto.response.* |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **clue / customer** | home → 业务模块 | HomeStatisticService 调 clue/customer Mapper.countXxx（可绕过 Service 直接查 Mapper，注意数据权限拼接） |
| **follow** | home → follow | "待跟进/已逾期"统计：HomeStatisticService 调 ExtFollowUpPlanMapper 按 owner + status 查 |
| **dashboard** | home ← dashboard（模块数据源） | 首页卡片与 Dashboard 模块可共享数据源；Dashboard 模块配置中可指定 home statistic 接口 |

---

## 六、常见边界场景

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **首页 10+ 统计接口并行调用 DB 压力大** | 首页所有 count 用 count(0) + 命中索引；可加短 TTL 本地缓存（10s）避免 F5 刷爆 DB；热点部署 Redis 缓存 | HomeStatisticService 方法前加本地缓存注解（如有） |
| **数据量百万级首页加载慢** | 百万级以上建议把日度统计结果写入汇总表（如 future 版本），或用 DataEase 离线同步 + Dashboard 展示；当前版本首页统计仅支持中小规模 | 集成 DataEase：home statistic 改从 DataEase 嵌入式报表查（integration.dataease） |
