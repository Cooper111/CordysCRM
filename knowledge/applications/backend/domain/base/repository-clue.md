---
type: domain-base
sub-type: repository-index
module: clue
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/clue/mapper/ 目录 + resources/mapper/clue/*.xml
owner: cordys-crm
verify-required: true
verify-note: 实际表名/字段/索引/数量 易变，核对 Flyway + Ext*Mapper.xml
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-clue.md
  - knowledge/applications/backend/domain/base/repository-core.md
---

# 线索模块 存储索引（repository-clue）

> 只写定位入口：Mapper 类 → 表前缀 → XML 位置 → Flyway 脚本位置。具体字段核对代码。

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀（实际名核对代码 @TableName） | XML 位置（resources/mapper/clue/） |
|---|---|---|---|
| **ExtClueMapper** | 线索主档 CRUD + 复杂查询（分页统计/列表带扩展字段 JOIN） | crm_clue | ExtClueMapper.xml |
| **ExtClueFieldMapper** | 线索扩展字段行存储 CRUD | crm_clue_field | ExtClueFieldMapper.xml |
| **ExtClueFieldBlobMapper** | 线索扩展 BLOB 存储 CRUD | crm_clue_field_blob | ExtClueFieldBlobMapper.xml |
| **ExtCluePoolMapper** | 线索池 CRUD + 按池查规则 | crm_clue_pool | ExtCluePoolMapper.xml |
| **ExtClueCapacityMapper** | 线索容量 查/改 | crm_clue_capacity | ExtClueCapacityMapper.xml |
| **ExtClueOwnerHistoryMapper** | 负责人变更历史 CRUD | crm_clue_owner_history | ExtClueOwnerHistoryMapper.xml |
| **ExtPoolCluePickRecordMapper** | 公海领取记录 | crm_pool_clue_pick_record（表名核对） | ExtPoolCluePickRecordMapper.xml |
| **ExtClueQuarantinedMapper** | 线索重复隔离期表 | crm_clue_quarantined（表名核对） | ExtClueQuarantinedMapper.xml |

> 通用表（operation_log / module_field / sys_dict / user_view 等）见 repository-core.md / repository-system.md，不重复。

---

## 二、Flyway 脚本位置

```
backend/crm/src/main/resources/db/migration/
  V{YYYYMMDDHHmm}__create_crm_clue_tables.sql        # 线索/线索池/容量/历史 首次建表（日期前缀不同）
  V{YYYYMMDDHHmm}__add_clue_field_blob.sql           # 扩展 BLOB 加表
  V{YYYYMMDDHHmm}__add_clue_quarantined.sql          # 隔离期表
```
（脚本名按日期不同，**不允许修改已运行脚本**——R5-B）

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 线索分页 SQL / 按权限过滤 / JOIN 扩展字段 | ExtClueMapper.xml 的 pageXxx / selectXxx 方法 |
| "我列表里多了个自定义字段怎么查出来的" | ExtClueMapper.xml 中 LEFT JOIN crm_clue_field 动态行转列（或 ClueFieldService 行转列逻辑，核对实现） |
| 线索池 批量投放 / 回收 SQL | ExtCluePoolMapper.xml |
| 线索容量 原子扣减/扣回 | ExtClueCapacityMapper.xml（注意 SQL 行锁 where ... for update / 乐观锁 version） |
| 操作日志 | 见 repository-system.md 的 operation_log |
