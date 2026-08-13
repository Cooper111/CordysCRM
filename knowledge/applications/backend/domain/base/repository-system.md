---
type: domain-base
sub-type: repository-index
module: system
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/system/mapper/（+ framework RBAC mapper）
owner: cordys-crm
verify-required: true
verify-note: 表名/字段/权限节点 易变，最大的模块（20+ 表），核对 Flyway
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-system.md
  - knowledge/applications/backend/domain/base/repository-core.md
---

# 系统管理 存储索引（repository-system）

> 系统模块表最多，覆盖 RBAC / 模块字段 / 字典 / 日志 / 配置 / 视图 / 调度。每类表独立 Ext*Mapper，本文件只写定位入口。

---

## 一、Mapper + 表定位（按分类）

### 1. RBAC（5 表）
| ExtMapper 类名 | 表前缀 | 职责一句话 |
|---|---|---|
| ExtUserMapper | sys_user | 用户 |
| ExtRoleMapper | sys_role | 角色 |
| ExtDeptMapper（或 ExtDepartmentMapper） | sys_dept | 部门树 |
| ExtUserRoleMapper | sys_user_role | 用户↔角色 多对多 |
| ExtRolePermissionMapper | sys_role_permission | 角色↔权限节点 |
| ExtPermissionMapper | sys_permission | 权限节点元数据（@CsPermission 扫描产物） |

### 2. 模块字段扩展（3 表）
| ExtMapper 类名 | 表前缀 | 职责一句话 |
|---|---|---|
| ExtModuleMapper | sys_module | 模块注册表（CLUE/CUSTOMER…） |
| ExtModuleFieldMapper | sys_module_field | 字段元数据 |
| ExtModuleFieldGroupMapper | sys_module_field_group | 字段分组 |
| ExtModuleFieldRolePermissionMapper（若独立） | sys_module_field_role_permission | 字段-角色可见/可编辑权限 |

### 3. 操作日志（2 表）
| ExtMapper 类名 | 表前缀 | 职责一句话 |
|---|---|---|
| ExtOperationLogMapper | sys_operation_log（可能按月份分表 _YYYYMM） | 操作日志主档 |
| ExtOperationLogBlobMapper | sys_operation_log_blob | 超长 diff / 参数 / 返回值 |
| ExtOperationLogDiffMapper（若独立） | sys_operation_log_diff | 字段级 diff |

### 4. 字典/系统参数/用户视图
| ExtMapper 类名 | 表前缀 | 职责一句话 |
|---|---|---|
| ExtDictionaryMapper（ExtDictMapper） | sys_dict / sys_dictionary_category | 字典分类 |
| ExtDictionaryItemMapper | sys_dict_item / sys_dictionary_item | 字典项 |
| ExtSystemConfigMapper | sys_system_config | 系统参数 |
| ExtUserViewMapper | sys_user_view | 筛选视图 |

### 5. 调度 / 导出任务 （其他系统能力表）
| ExtMapper 类名 | 表前缀 | 职责一句话 |
|---|---|---|
| ExtScheduleMapper | sys_schedule | 自定义调度任务（业务级 Cron） |
| ExtExportTaskMapper | sys_export_task（或 operation_log_blob 里存，核对） | 导出任务 |
| ExtImportTaskMapper（如有） | sys_import_task | 导入任务 |

---

## 二、Flyway 脚本位置

通常拆多个脚本：
```
db/migration/V{ver}__create_rbac_tables.sql
db/migration/V{ver}__create_module_field_tables.sql
db/migration/V{ver}__create_operation_log_tables.sql
db/migration/V{ver}__create_dict_system_config_tables.sql
db/migration/V{ver}__create_schedule_tables.sql
```
不允许修改已运行脚本（R5-B）

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 某角色有哪些权限 → 授权 SQL | ExtRolePermissionMapper.xml |
| 刷新权限（扫描 @CsPermission 写 sys_permission 表） | PermissionMapper 的 upsert 逻辑 + CsPermissionScanner（framework 层） |
| 操作日志按月分表（表名 _202608 / _202609） | OperationLogMapper 动态表名逻辑（Sharding-JDBC 或手写 interceptor，核对实现） |
| 模块字段 行转列 动态查询 | ModuleFieldService + 各业务 Ext{Module}Mapper.xml 动态拼接 |
| "字典变了下拉没变" → 字典缓存 | DictionaryService cache evict 逻辑 + Redis key |
