---
type: domain-base
sub-type: api-index
module: system
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/system/controller 目录
owner: cordys-crm
verify-required: true
verify-note: 权限节点、模块字段扩展、操作日志 接口参数易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/system.md
---

# 系统管理 模块 API 索引（api-system）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **SysUserController** | 用户 CRUD、重置密码、启用/停用、绑定角色、导入导出用户 | UserService（或 SysUserService） + UserRoleService | `SYSTEM:USER_ADD/EDIT/DELETE/VIEW/IMPORT/EXPORT/RESET_PASSWORD` |
| **SysRoleController** | 角色 CRUD、绑定权限节点、绑定用户、权限克隆、角色启用/停用 | RoleService + RolePermissionService + UserRoleService | `SYSTEM:ROLE_ADD/EDIT/DELETE/VIEW/BIND_PERMISSION/CLONE` |
| **SysDeptController**（如有独立） | 部门树 CRUD、部门排序、部门启用/停用、部门负责人设置 | DeptService | `SYSTEM:DEPT_ADD/EDIT/DELETE/VIEW` |
| **PermissionController** | 权限节点查询（整棵权限树）、权限节点刷新（扫描 @CsPermission 注解写入数据库） | CsPermissionService（框架） | `SYSTEM:PERMISSION_MANAGE` |
| **ModuleFieldController** | 模块字段扩展：按 moduleKey 查询字段列表、字段 CRUD、字段显示/隐藏、字段必填、字段默认值、字段分组、字典关联 | ModuleFieldService + ModuleFieldValidatorService | `SYSTEM:MODULE_FIELD_MANAGE` |
| **OperationLogController** | 操作日志查询（列表/详情/导出）、按模块/操作人/时间过滤 | OperationLogService（由 ModuleLogServiceFactory 产生各模块 Service 实例） | `SYSTEM:OPERATION_LOG_VIEW/EXPORT` |
| **SystemConfigController**（如有） | 系统参数配置（如线索回收超时天数、商机阶段配置开关、容量默认值） | SystemConfigService | `SYSTEM:CONFIG_MANAGE` |
| **UserViewController** | 用户筛选视图（通用，按 moduleKey）：保存我的筛选条件、视图 CRUD、视图公开/私有切换 | UserViewService | `SYSTEM:USER_VIEW_ADD/EDIT/DELETE` |
| **DictionaryController**（如有） | 字典项/字典分类 CRUD、排序、启用/停用 | DictionaryService + DictionaryItemService | `SYSTEM:DICT_MANAGE` |
| **CsPermissionRefreshController**（如有） | 权限注解扫描工具入口：重新扫描 @CsPermission 刷新 permission 表 | CsPermissionService.scanAndSync() | `SYSTEM:PERMISSION_REFRESH` |

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 用户管理 / 重置密码 / 导入用户 | SysUserController |
| 角色 / 权限分配 / 角色克隆 | SysRoleController |
| 部门树 | SysDeptController（核对是否独立） |
| 权限树 / 刷新权限 | PermissionController / CsPermissionRefreshController |
| 模块字段扩展（某模块新增自定义字段） | ModuleFieldController |
| 操作日志 / 谁改了什么 / 导出日志 | OperationLogController |
| 系统参数 / 配置项 | SystemConfigController（核对是否有） |
| 列表保存筛选条件视图 | UserViewController（通用） |
| 字典维护（下拉选项） | DictionaryController（核对） |

路径前缀（核对 @RequestMapping）：
```
/api/sys-user/*              → 用户
/api/sys-role/*              → 角色
/api/sys-dept/*              → 部门
/api/permission/*            → 权限
/api/module-field/*          → 模块字段
/api/operation-log/*         → 操作日志
/api/system-config/*         → 系统参数
/api/user-view/*             → 筛选视图（通用）
/api/dictionary/*            → 字典
```
