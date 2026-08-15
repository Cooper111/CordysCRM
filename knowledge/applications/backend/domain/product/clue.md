---
type: domain-product
app: backend
module: clue
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/clue 下 Controller + Service + Domain + Mapper（共 9 Controller / 9 Service / 8 Domain / 4 Mapper）
owner: cordys-crm
verify-required: true
verify-note: 接口签名、枚举值、DTO 字段必须回代码核对（易变项 R4）
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#线索/线索池/线索容量
  - knowledge/main/core-process.md#2-线索生命周期
  - knowledge/main/state-defs.md#一线索状态
  - knowledge/main/business-rules.md#2-线索/客户容量规则 + #3-线索池/客户池公海规则
  - knowledge/applications/backend/domain/base/api-clue.md
  - knowledge/applications/backend/domain/base/domain-clue.md
  - knowledge/applications/backend/domain/base/repository-clue.md
---

# 线索模块 · 产品能力

---

## 一、能力概述

解决"潜在客户（线索）"从**录入、分配、跟进、转化为客户**到最终**回收进入公海再分配**的全生命周期管理，确保销售线索的高效流转、容量公平、分配透明、跟进可追溯。

主干流程：
```
线索录入（手工/导入/分配） → 分配或领取给销售 → 跟进（计划+记录） → 转客户成功
  ↑                                                          ↓ 失败/未跟进
  └─── 线索池回收（定时自动或手动退回） ←── 容量释放/超时 ───┘
```

---

## 二、核心流程（ASCII 流程图）

```
 ┌──────── 录入方式 ─────────┐
 │ 新建(单条) / 导入(批量)   │
 └──────────┬───────────────┘
            │ 写入 clue 表 + ClueField
            ▼
   [状态：新建] ──分配(PoolClueService.assign) / 销售领取──▶ [状态：已分配]
            │                                              │
            │ 未分配超期 (CluePoolRecycleListener)         │ 销售跟进
            ▼                                              │  FollowUpPlanService.createForClue
   [入线索池] ◀──手动退回线索池/容量超限回收──────── [状态：跟进中]
   (CluePoolService)                                       │
            │ 销售领取 PoolClueService.pick                │ 满足条件
            ▼                                              ▼
   [循环]                                     [线索转客户 → CustomerService.transformFromClue]
                                                      │
                                                      ▼
                                               创建 Customer + Contact
                                               同时原线索状态：已转化
                                               (非终态，线索记录仍保留查询)
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 涉及代码入口（定位类名+方法，细节回代码核对） |
|---|---|---|---|
| 线索录入 | 销售新建 / 后台导入 / 公海分配 | 字段校验 + 负责人写入 + 操作日志 + 跟进计划(可选) | `ClueService.add()` 写主表+字段表；ClueExportService 导入（system/ExportTask 异步） |
| 线索分配/领取 | 管理员批量分配 / 销售从公海领取 | **容量校验**（ClueCapacityService.check）→ 超过容量拒绝；写入 ClueOwner 历史；操作日志 | `CluePoolService.assignToUsers()` 分配；`PoolClueService.pickClue()` 销售领取 |
| 线索跟进 | 销售按计划跟进 | 跟进计划（FollowUpPlanService）+ 跟进记录（FollowUpRecordService）+ 关联 @提醒（Notice） | `ClueFollowPlanController` / `ClueFollowRecordController` 代理 follow.*Service |
| 线索转客户 | 线索足够成熟点击"转客户" | 校验客户名称/联系人不为空 → 查重（CustomerContactService）→ 写 Customer + Contact → 状态改为"已转化"；失败回滚事务 | `ClueService.transformToCustomer()` → 调用 `CustomerService.transformFromClue()`；事务边界见 ClueService @Transactional |
| 线索池回收 | 跟进超时未续 / 未跟进超期 / 容量超限 / 手动退回 | 定时任务 CluePoolRecycleListener → 按 CluePoolRecycleRule 批量查 → 调用 CluePoolService.recycleOne，写回 clue_pool，清空负责人 | `CluePoolRecycleListener`（Quartz 触发）；`CluePoolService.recycleBatch()` |
| 导入导出 | 点击导入/导出按钮 | 导出：ClueExportService（大导出走异步 → system/ExportTask）；导入：system/ImportService → 校验→写 Clue | `ClueExportService` + `ImportType.CLUE`（system.constants） |

---

## 三、核心业务规则（判断条件，非实现细节）

| 规则 ID | 规则描述（if/then 判断） | 例外情况（谁可以绕过） |
|---|---|---|
| R1 | **容量规则**：分配/领取前 if (当前负责人.线索数 >= 角色容量上限) then 拒绝操作 | 超级管理员（RoleId=1）强制分配会忽略容量检查，但依然记录超容 |
| R2 | **转客户条件**：if (线索企业名称为空 OR 联系人为空) then 拒绝转化为客户 | 管理员可通过特定开关强制转化（需记录操作日志说明"无企业名称强制转化"） |
| R3 | **分配去重**：if (线索已分配给该用户 OR 近 N 天领取过) then 拒绝重复领取 | 公海管理员（POOL_ADMIN 权限）可通过"再次分配"绕过 N 天限制 |
| R4 | **公海可见性**：线索池内 if (字段被 CluePool.hiddenFields 隐藏) then 对领取用户展示 **（仅管理员可见原值，销售不可见）** | 公海管理员 / 系统管理员查看池内信息依然可见全字段 |
| R5 | **字段权限**：线索自定义字段（ModuleField）if (该用户无字段查看权限) then 返回脱敏或 null | 导出时按字段权限裁剪列，和列表一致 |
| R6 | **线索 owner 历史**：每次线索负责人变更时 if (写入 ClueOwnerHistory) then 记录变更前后人、变更时间、变更原因 | — |
| R7 | **并发更新乐观锁**：同一线索 if (两个用户同时编辑) → 采用"后写覆盖+操作日志记录"策略，不做版本字段冲突检测 | 重要字段（负责人、状态、企业名称）变更必须记录操作日志前后 diff（由 ModuleLogServiceFactory 代理 ClueLogService） |
| R8 | **批量操作权限**：批量编辑/批量转客户前 if (batch 中有 1 条无权限) then 整批拒绝（CsBatchPermission 切面），部分成功不允许 | 管理员（ROLE_ADMIN）默认通过 |

---

## 四、状态/枚举（**语义定位，实际值回代码核对**）

| 语义 | 枚举类名 / 常量名（**定位入口，实际编码值核对代码**） | 代码入口路径 |
|---|---|---|
| 线索状态（新建/已分配/跟进中/已转化/已回收...） | `cn.cordys.crm.clue.constants.ClueStatus` 枚举 | backend/crm/src/main/java/cn/cordys/crm/clue/constants/ClueStatus.java |
| 公海领取规则（按权重/按轮询/按数量上限） | `cn.cordys.crm.clue.domain.CluePoolPickRule` 字段 type | clue.domain.CluePoolPickRule.java |
| 公海回收规则类型（未跟进N天/已分配N天） | `cn.cordys.crm.clue.domain.CluePoolRecycleRule` 字段 type | clue.domain.CluePoolRecycleRule.java |
| 模块权限节点（线索） | `cn.cordys.common.constants.PermissionConstants` 中 CLUE: 前缀 | common/constants/PermissionConstants.java（易变需核对最新） |
| 表单 key（线索） | `FormKey.CLUE`（线索）、`FormKey.CLUE_POOL`（线索池） | common/constants/FormKey.java |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约（接口/服务/DTO/事件） |
|---|---|---|
| **customer（客户）** | clue → customer | 转客户调用 `CustomerService.transformFromClue(ClueTransformRequest)`，传 Clue 原始数据 |
| **customer（客户）** | clue ← customer | 客户反转化（还原线索）调用 `ClueService.reTransformFromCustomer()`，恢复线索状态 |
| **follow（跟进）** | clue → follow | 线索跟进计划/记录：ClueFollowPlanController 调用 follow.*.Service，resourceType=CLUE，resourceId=clue.id |
| **opportunity（商机）** | clue → opportunity | 转客户时可自动创建商机（可选），传 `OpportunityAddRequest` |
| **approval（审批）** | clue ← approval | 某些场景下线索分配/转客户需要审批，审批回调更新状态（approval.handler.ApprovalResourceHandler 扩展） |
| **system（导出/导入/日志）** | clue → system | 导出写 ExportTask（ExportConstants.ExportType.CLUE）；操作日志通过 `ClueLogService extends BaseModuleLogService`；字典 DictModule.CLUE |
| **system（用户视图）** | clue → system | ClueUserViewController 委托 UserViewService 保存/查询列表筛选视图 |
| **system（容量）** | clue → system | 容量数据存 ClueCapacity，角色容量来自 OrganizationConfig + Parameter |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口类（定位） |
|---|---|---|
| **重复线索（相同企业名+手机号）** | 录入/转客户前查重：ClueService.add 中通过条件查询 Clue + CustomerContact 联合比对，命中则提示"是否合并/已存在客户" | ClueService.add / CustomerContactService.checkRepeat |
| **导入文件量大（1W+行）** | 拒绝同步导入：走 system/ImportService 异步任务 + Import 进度；导入失败的行写回错误文件 | ExportTask / ImportRequest + ImportType.CLUE |
| **大导出 OOM（10W+ 线索）** | 导出拆分页 + 流式写（SXSSF）+ 超阈值自动异步，下载通过 ExportTask 导出任务中心 | ClueExportService（实现模式参考 tech/excel-io.md） |
| **池字段隐藏但销售截图已泄露** | 策略：隐藏字段仅在领取**前**隐藏；一旦被销售领取 → 自动解锁全部字段（写入 Clue 后不再隐藏） | PoolClueService.pickClue → 取 PoolClueFieldUtils.mergeFieldConfig |
| **负责人离职（批量移交）** | 通过 ClueOwnerHistory + 批量移交接口：校验移交目标容量 → 逐线索变更 owner → 写历史 → 发通知 | ClueService.batchTransfer（ClueBatchTransferRequest） |
| **销售权限被回收后线索归属** | 线索不自动回收：仍挂在原负责人名下（可查不可改，或由主管手动移交）；通过 CsPermission 写切面拦截 | CsPermission 切面 + 权限节点 CLUE:EDIT |
| **转客户后线索数据需要保留** | 线索保留原表，状态=已转化（逻辑保留）；查询/导出默认仍展示（可过滤），删除需独立 RECYCLE 接口 | Clue 表 status 字段 + ClueGetResponse 保留映射 |
| **操作日志的前后 diff 不准确（自定义字段）** | diff 委托 ClueFieldService 获取字段完整值后比较，按 ModuleField 的 displayName 展示中文列名 | ClueLogService extends BaseModuleLogService + JsonDifferenceDTO |
