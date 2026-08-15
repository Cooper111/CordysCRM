---
type: domain-product
app: backend
module: system
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/system 下 20+ 个领域类（用户/角色/部门/组织/模块/模块字段/字典/公告/参数/操作日志/导出任务/调度/用户视图/个人中心/组织配置/消息通知/版权）+ 15+ 个 Controller / 30+ Service / 20+ Mapper
owner: cordys-crm
verify-required: true
verify-note: 权限节点、参数字典、表数量大，所有易变项（节点/参数键/字段类型编码）必须回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#权限/角色/部门/组织/操作日志/公海
  - knowledge/main/business-rules.md（全部章节：权限/容量/公海/审批/日志/通知/多租户）
  - knowledge/main/tech-constraints.md（后端约束）
  - knowledge/applications/backend/domain/base/api-system.md
  - knowledge/applications/backend/domain/base/domain-system.md
  - knowledge/applications/backend/domain/base/repository-system.md
  - knowledge/applications/backend/tech/security.md + operation-log.md + schedule-job.md + common-pitfalls.md
---

# 系统管理模块 · 产品能力

---

## 一、能力概述

系统管理模块是 CRM 的"底座"，覆盖全部"基础设置"能力，是所有业务模块的依赖。拆分为 9 大子域：
1. **用户体系**：用户 / 角色 / 部门 / 组织 / 用户角色关联 / 角色权限 / 数据范围
2. **模块与字段**：模块管理 / 模块字段 / 模块表单 / 字典
3. **操作审计**：操作日志 + 模块日志工厂（各业务模块 extend BaseModuleLogService）
4. **导入导出任务中心**：导出（异步大导出） + 导入（异步校验入库）
5. **系统参数与公告**：Parameter（系统参数键值对）+ Announcement（公告）
6. **用户视图**：各业务模块的筛选视图（UserView + UserViewCondition）
7. **调度中心**：Quartz 定时任务（qrtz_*）+ Schedule 自定义任务 + 监听器（线索/客户回收、跟进提醒、规则引擎等）
8. **个人中心**：PersonalCenter（改资料/改密码/看自己的跟进/发邮件/版权信息）
9. **组织配置与通知**：多租户 OrganizationConfig（组织级开关：审批是否启用、公海回收规则开关等）+ 消息通知（站内信/邮件/IM）+ 模板

主干："业务模块 → 依赖 system → 完成权限/日志/字段/导入导出/通知等通用能力"

---

## 二、核心流程（ASCII 流程图，9 大子域关键链路）

```
  ┌───────────────── 1. 用户体系 ──────────────────┐
  │ 新建角色 → 绑定权限节点(permission.json) +     │
  │   数据范围(RoleScopeDept) → 分配用户（UserRole）│
  │ 新建部门 → 部门树（Department 父子结构 + 排序） │
  │ 新建用户 → 绑定部门 + 角色 + 初始化密码/SSO登录 │
  │            → UserExtend 扩展字段                │
  │          CsPermission + DataScopeService 生效    │
  └────────────────────────────────────────────────┘
                    ▲
  ┌───────────────── 2. 模块字段 ───────────────────┐
  │ Module（业务模块 CRUD + 启用/停用 + 菜单排序）    │
  │   ↓ ModuleField：业务模块自定义字段（类型/必填/唯一│
  │       / 字段来源：字典/用户/部门/级联/文本/数字…  │
  │   ↓ ModuleForm：字段布局/分组（前端渲染配置）     │
  │   ↓ Dict：数据字典 + 字典项 + 级联字典            │
  └────────────────────────────────────────────────┘
                    ▲
  ┌───────────────── 3. 操作审计 ───────────────────┐  ┌──────── 6. 用户视图 ────────┐
  │ @OperationLog 注解 → AspectJ 切面拦截写操作      │  │ UserView：按业务资源类型保存│
  │ → LogDTO → ModuleLogServiceFactory 路由到        │  │   我的筛选视图/公开视图     │
  │   业务 LogService（ClueLog/CustomerLog…）→      │  │ UserViewCondition：一组条件│
  │   写 operation_log + operation_log_blob（JSON    │  │   (JSON 数组，ConditionDTO)│
  │   diff 前后字段值对比，供操作日志详情展示）        │  │  业务模块的 XxxUserViewController
  └────────────────────────────────────────────────┘  └─────────────────────────────┘
                    ▲
  ┌───────── 4. 导入导出任务中心 ─────────┐    ┌─────────── 5. 参数/公告 ───────────┐
  │ 导出：ExportTask + 异步写入 / 下载链接 │    │ Parameter：系统参数键值对          │
  │      → 各业务 ExportService 分页+流式 │    │  例：导出阈值、默认容量、开关等     │
  │      → 失败标记 / 成功下载次数         │    │ Announcement：公告（富文本）       │
  │ 导入：ImportRequest + ImportType 枚举 │    │  新建/发布/置顶/过期               │
  │      → 校验 → 批处理入库 → 错误行文件 │    └────────────────────────────────────┘
  └────────────────────────────────────────┘
                    ▲
  ┌───────────────── 7. 调度中心 ────────────────────┐
  │ Schedule（自定义定时任务表）+ Quartz（qrtz_*表）   │
  │   Job 示例：                                       │
  │   ├ CluePoolRecycleListener：线索池回收            │
  │   ├ CustomerPoolRecycleListener：客户池回收        │
  │   ├ OpportunityRuleListener：商机规则执行          │
  │   ├ FollowUpPlanRemindListener：跟进计划提醒       │
  │   ├ NoticeExpireJob：过期通知清理                  │
  │   ├ CleanTempResourceListener：清理临时文件/资源   │
  │   ├ CleanExportResourceListener：清理过期导出      │
  │   └ TaskCleanupJob / SessionJob：任务/会话清理     │
  │   Quartz 支持手动触发（ScheduleController.trigger）│
  └────────────────────────────────────────────────────┘
                    ▲
  ┌────────── 8. 个人中心 ──────────┐    ┌──────── 9. 组织配置/通知 ────────┐
  │ PersonalCenterService：         │    │ OrganizationConfig：组织级开关+配置│
  │  - 修改资料/头像/密码            │    │  (审批启用、容量默认值、公海配置…) │
  │  - 我的跟进计划列表              │    │ CommonNoticeSendService：通知统一入口│
  │  - 发送邮件 (MailSender)        │    │  站内信 + 邮件 + 企微/钉钉/飞书IM │
  │  - 版本版权信息 (CopyrightUtils)│    │  模板：MessageTemplateUtils + 参数替换│
  └──────────────────────────────────┘    └────────────────────────────────────┘
```

### 步骤详解（核心 6 条，其余核对代码）

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 写操作日志切面 | 任意 Controller 写操作方法带 `@OperationLog` | aspectj → OperationLogContext 挂上下文 → Service 方法返回后 → ModuleLogServiceFactory 按 LogModule 路由到对应业务 LogService → 写 operation_log + operation_log_blob | `cn.cordys.aspectj` 下切面 + crm.system.service.ModuleLogServiceFactory（按 LogModule 枚举路由到 ClueLog/CustomerLog...） |
| 权限校验（功能权限） | 任意 Controller 请求 | CsPermissionAspect → 方法 @CsPermission("NODE:ACTION") → Shiro hasPermission → PermissionCache → 权限定义来自 resource/permission.json（易变核对代码） | common.permission.CsPermissionAspect + PermissionCache |
| 数据范围过滤 | 任意业务列表查询 | Service 层在 base QueryWrapper 前 → DataScopeService.injectScope(当前用户, 资源类型) → 拼 SQL 条件（SELF/DEPARTMENT/ALL/CUSTOMER_COLLABORATION/ROLE_SCOPE_DEPT） | common.service.DataScopeService + DeptDataPermissionDTO |
| 异步大导出 | 业务模块点"导出"且数据量 > 阈值 | 业务 ExportService → system.ExportTaskService：创建 ExportTask 记录 → 后台线程池分页查 + SXSSF 流式写 → 任务完成后站内信通知用户 + 下载链接 7 天过期 | crm.system.service.ExportTaskService + CleanExportResourceListener |
| Quartz 监听器 + Schedule 手动触发 | 定时触发 / 管理员后台点"立即执行" | Quartz Job 触发对应 QuartzJobBean → 内部调用监听器（Spring ApplicationListener 风格）；管理后台 ScheduleController.trigger 通过 Scheduler 立即 fire 一次 | crm.system.job.* + crm.system.job.listener.* + ScheduleService |
| 统一通知发送 | 跟进/审批/回收/分配… 各种触发点 | 代码内调用 CommonNoticeSendService.send(NoticeSendRequest)：按用户的 NotificationConstants.XXX 配置的渠道（站内/邮件/IM）→ 发站内信写 DB + 发邮件 MailSender + 调 LarkNoticeSender / DingTalkNoticeSender / WeComNoticeSender（按用户配置，按实现核对） | system.notice.CommonNoticeSendService + integration.*.NoticeSender 实现 |

---

## 三、核心业务规则（全局通用，重点提炼 10 条）

| 规则 ID | 规则描述 | 例外 |
|---|---|---|
| R1 | **用户停用 = 所有写权限立刻失效**：if (User.status = DISABLED) then CsPermissionAspect 直接拦截写操作，仅允许少量只读（查自己的个人中心） | 管理员可临时启用 |
| R2 | **超管角色（RoleId=1）不可删除、不可改名为非管理员**：硬编码保护，防止误操作锁死系统 | — |
| R3 | **密码长度/强度**：密码修改前 if (长度<8 OR 不满足复杂度) then 拒绝；登录失败 5 次锁定 30 分钟（具体值按参数核对） | 管理员重置密码不受强度限制（但建议用户立刻改） |
| R4 | **模块字段删除不物理删**：删除模块字段 = 标记逻辑停用 → 已保存历史数据的 Field Blob 仍保留旧字段值（展示时标记已废弃）；如需物理清走离线脚本清理（半年一次） | 超管有"彻底清字段+所有历史数据"按钮（危险操作，需二次确认+手机验证） |
| R5 | **操作日志永久保存**：operation_log 和 operation_log_blob 不提供清理界面；如需合规清理 5 年前日志 → DBA 归档脚本按 created_at 范围移到归档库 | — |
| R6 | **导出 7 天自动清理**：ExportTask 中 created_at > 7 天的文件，CleanExportResourceListener 每日 0 点删除本地文件 + 标记记录=过期 | 管理员可在后台点击"延期 7 天" |
| R7 | **字典项被引用不可删**：if (字典项被模块字段默认值/业务表存储引用) → 删除字典项时拒绝或告警 | 超管强制删 + 选择"替换默认值为 xxx"迁移 |
| R8 | **参数 Parameter 键唯一**：if (同组织 param.key 重复) 拒绝；参数 key 命名约定 `模块.子项.项`（例：clue.capacity.default） | — |
| R9 | **用户视图公开 = 全组织可见**：UserView.isPublic=true → 所有人只读可看；可复制成"我的视图"后编辑；编辑/删除只允许 owner 或超管 | — |
| R10 | **跨组织数据严格隔离**：所有 system / crm 表有 org_id（或 OrganizationContext） → 所有查询 SQL 都被 OrganizationAspect 自动注入 org_id 过滤；超管也不能跨组织查 | 超管跨组织操作 → 先在"组织切换"（如有）切换组织身份才可以 |

---

## 四、状态/枚举/常量（定位入口，最常查）

| 语义 | 枚举/类名入口 | 代码路径 |
|---|---|---|
| 业务模块（clue/customer/opportunity/contract/order/product/follow/form/approval） | `ModuleKey` 枚举（common.constants.ModuleKey）+ `BusinessModuleField` | common.constants.ModuleKey.java / BusinessModuleField.java |
| 表单 Key（对应业务对象的表单） | `FormKey` 枚举（FORM:CLUE / CUSTOMER / CONTRACT / OPPORTUNITY / ORDER / PRODUCT / FOLLOW_UP_PLAN / RECORD / BUSINESS_TITLE / CUSTOM_FORM / CUSTOM_FORM_DATA …） | common.constants.FormKey.java |
| 权限节点（CsPermission 注解值） | `PermissionConstants` 常量（CLUE:*、CUSTOMER:*、APPROVAL:*、SYSTEM:*…）+ resource/permission.json（实际定义） | common.constants.PermissionConstants.java + backend/app/src/main/resources/permission.json |
| 字典模块 | `DictModule` 枚举（线索来源、客户阶段、商机阶段配置、跟进类型等字典分类标识） | crm.system.constants.DictModule |
| 导入类型 | `ImportType` 枚举（CLUE / CUSTOMER / OPPORTUNITY / CONTRACT / ORDER / PRODUCT / CUSTOM_FORM_DATA） | crm.system.constants.ImportType |
| 导出类型 | `ExportConstants` + ExportType（同 ImportType） | crm.system.constants.ExportConstants |
| 通知类型常量（站内信/邮件/IM） | `NotificationConstants`（CLUE_ASSIGNED / CUSTOMER_RECYCLE / FOLLOW_UP_REMIND / APPROVAL_PENDING / APPROVAL_PASS / APPROVAL_REJECT…） | crm.system.constants.NotificationConstants.java |
| 字段类型 | `FieldType` 枚举（TEXT / TEXTAREA / NUMBER / DATE / DATETIME / DICT_SINGLE / DICT_MULTI / USER / DEPARTMENT / CASCADE / ATTACHMENT / REFERENCE…） | crm.system.constants.FieldType |
| 字段来源类型 | `FieldSourceType`（普通/字典/用户/部门/级联/业务资源引用…） | crm.system.constants.FieldSourceType |
| 视图资源类型 | `UserViewResourceType` 枚举（CLUE / CUSTOMER / OPPORTUNITY…） | crm.system.constants.UserViewResourceType |
| 组织配置键 | `OrganizationConfigConstants`（审批开关/默认容量/回收规则开关…） | crm.system.constants.OrganizationConfigConstants.java |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **所有业务模块（clue/customer/opportunity/contract/order/product/follow/form/approval/home/dashboard）** | 业务 → system | **所有业务模块都强依赖 system**：权限 CsPermission、数据范围 DataScope、操作日志 @OperationLog、模块字段 ModuleField、字典 Dict、导入导出 Import/Export、视图 UserView、参数 Parameter、通知 Notice、调度 Schedule；业务模块独立 system 无法运行 |
| **framework（权限 Shiro / Mybatis BaseMapper / UID / AspectJ / Redis / Session）** | system → framework | Shiro Realm / Session（Redis）/ UIDGenerator（WorkerNode）/ BaseMapper 继承 / 切面注解实现 都定义在 framework；system 提供业务层包装 Service |
| **integration（钉钉/飞书/企微/SSO/企查查/DataEase/MaxKB）** | system → integration | 统一通知 = 调集成 NoticeSender（Lark/DingTalk/WeCom）；第三方部门同步 = sync 包 + ThirdDepartmentService；SSO登录 = sso 包；第三方配置 = integration.common 下 XXXThirdConfigRequest |
| **approval（审批）** | approval → system | 审批通过/驳回 通知调 system.notice；审批资源处理器写回业务状态 → 操作日志调 system ModuleLogService；审批定义按 org 隔离来自 OrganizationContext（framework.context） |
| **个人中心 vs 业务 follow** | personal-center → follow | 我的跟进计划/记录列表 = PersonalCenterService → 直接调 ExtFollowUpPlanMapper（绕过 follow Service 做过滤优化，不影响逻辑） |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **忘记超管密码** | 数据库执行密码重置脚本（由部署文档提供），或 installer 内置"重置超管密码.sh"脚本 | 不在 system 模块内暴露接口，避免被利用 |
| **CsPermission 节点定义在 permission.json 但后端没生效** | 权限节点需要满足两个条件：① 在 permission.json 中定义结构；② 在 Controller 方法上有 @CsPermission("XXX") 注解；两者缺一不可；少一个导致权限不生效或 403 | 每次加新权限节点必须同时更新 json + 注解 |
| **导入失败数据量很大，错误行 Excel 生成 OOM** | 错误文件：用 EasyExcel 流式写，边校验边记录；10W+ 错误行也不把全部错误放内存，而是边校验边写临时文件最后汇总 | Import 服务实现（核对代码 ImportService） |
| **操作日志量太大（1000W+）DB 膨胀** | ① operation_log 表按月分表（如已有则核对 Flyway）；② 定期归档到 history 库；③ 详情页查询 operation_log_blob 时只查 6 个月内的，更早走归档查询接口 | 部署文档或 Schedule 归档任务 |
| **通知 IM 失败（第三方服务超时）** | CommonNoticeSendService：第三方推送超时 10s，不阻塞业务方法 → 失败仅记录日志 + 站内信保底送达；IM 重试 2 次仍失败则放弃 | 各 NoticeSender 实现（超时配置 + 失败重试） |
| **部门树形层级过深（15+）导致 countScopeDept 慢** | 部门查询路径：DepartmentService 用"闭包表或路径字符串"做快速全子部门查询；DataScopeService 注入 DEPARTMENT 范围时直接 IN(deptIds)，避免 SQL 递归 | DepartmentService 按实现核对 |
