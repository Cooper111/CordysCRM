---
type: domain-base
sub-type: repository-index
module: core
app: backend
status: official
evidence: backend/common/src/main/java/cn/cordys/mybatis/ + backend/crm/src/main/resources/db/ 目录
owner: cordys-crm
verify-required: true
verify-note: 表结构/字段/索引/版本号 易变，本文件只写定位入口，实际以 Flyway 脚本 V*_\_\_*.sql 为准
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/repository-clue.md
  - knowledge/applications/backend/domain/base/repository-system.md
---

# 存储层核心规范 索引（repository-core）

> 本文档是所有 repository-*.md 的父索引：定义本项目**通用 Mapper 用法/通用表结构/Flyway 规范/UID 生成**约定。
> 具体业务表见各模块 repository-{module}.md。

---

## 一、BaseMapper 用法定位

| 类名 | 职责一句话 | 包路径 | 关键特性 |
|---|---|---|---|
| **BaseMapper<T>** | 通用 Mapper 基类（MyBatis-Plus 封装） | cn.cordys.mybatis.BaseMapper | 默认 CRUD、分页 page()、批量 insertBatch、逻辑删除（配合 @LogicDelete） |
| **Ext*Mapper**（各模块实际 Mapper） | 业务扩展 Mapper：业务自定义 SQL 全写这里（XML 或注解） | cn.cordys.crm.{module}.mapper.* | 继承 BaseMapper<T>，禁止直接把复杂 SQL 写 Service 里 |
| **QueryWrapperBuilder / LambdaQueryUtil**（核对类名） | 查询条件构建工具类：防空传、时间范围、数据范围自动追加 | cn.cordys.mybatis.util.* | DataScope 权限字段拼接 |

### Ext*Mapper 的 XML 位置
```
backend/crm/src/main/resources/mapper/{module}/Ext{Module}Mapper.xml
如：backend/crm/src/main/resources/mapper/clue/ExtClueMapper.xml
```

---

## 二、通用表结构（系统全局表，非业务模块表）

| 表名（前缀，实际名核对 Flyway） | 职责一句话 | 对应 ExtMapper | 关联文档 |
|---|---|---|---|
| sys_operation_log（或 operation_log + 分表后缀 _YYYYMM） | 操作日志主档（ModuleLogServiceFactory 产出） | ExtOperationLogMapper | repository-system.md |
| sys_operation_log_blob | 操作日志 BLOB（diff JSON / 请求参数 超长） | ExtOperationLogBlobMapper | repository-system.md |
| sys_module | 模块注册（CLUE/CUSTOMER…） | ExtModuleMapper | repository-system.md |
| sys_module_field | 模块字段元数据（ModuleField） | ExtModuleFieldMapper | repository-system.md + domain-system.md |
| sys_module_field_blob | 模块字段扩展 blob | ExtModuleFieldBlobMapper | repository-system.md |
| sys_dict + sys_dict_item | 字典分类 + 字典项 | ExtDictMapper / ExtDictItemMapper | repository-system.md |
| sys_schedule | 自定义调度任务（业务级 Cron） | ExtScheduleMapper | repository-schedule.md |
| qrtz_job_details / qrtz_triggers / qrtz_* | Quartz 内建表（Sqrtz 框架表） | 无 Mapper（JDBC 访问） | repository-schedule.md |
| sys_permission / sys_role / sys_user / sys_user_role / sys_role_permission | RBAC 五表 | Ext*Mapper 多个 | repository-system.md |
| sys_user_view | 用户筛选视图（所有模块通用） | ExtUserViewMapper | repository-system.md |
| sys_system_config | 系统参数配置 | ExtSystemConfigMapper | repository-system.md |

---

## 三、Flyway 规范

| 约定 | 说明 | 定位位置 |
|---|---|---|
| 脚本命名 | `V{YYYYMMDDHHmm}__{description}.sql`（下划线 2 个） | backend/crm/src/main/resources/db/migration/ 下 |
| 基准版本 | V1__init.sql（首次完整建表，核对是否存在） | db/migration/V1__init.sql |
| 热修复脚本 | `V{最新版本号后面追加}__hotfix_{date}_xxx.sql`（不允许修改已运行的 V 脚本！Flyway 校验和冲突 R5-B） | 同目录 |
| 回滚 | **禁止改已有 V 脚本**；如需回滚，写新脚本 `V{ver+1}__rollback_xxx.sql` 做反向 DDL | 同目录 |
| 小版本数据修复 | `R__recheck_{date}_datafix.sql`（Repeatable，每次启动执行，脚本内必须幂等） | db/migration/ 下 R 开头 |

**严禁修改已合入的 V*.sql / R*.sql 脚本**（R5-B 底线）。

---

## 四、UID 生成

| 类/接口名 | 说明 | 包路径 |
|---|---|---|
| **IDGenerator**（或 SnowflakeIdWorker） | 全局唯一 ID 生成：雪花算法，返回 Long，业务主键一律用这个（不用自增 ID） | cn.cordys.common.uid.IDGenerator |
| **CodeGenerator**（单号生成） | 业务单号：QS{年月日}{6 位自增}、XS{yyyyMMdd}{6 位}、HT{yyyyMMdd}{6 位}… 在各模块 Service 中调用时传模块前缀 | cn.cordys.common.code.CodeGenerator（核对类名） |

定位：任何需要 newId 的地方，**不要用 UUID / 自增 / 时间戳**，一律用 IDGenerator。
