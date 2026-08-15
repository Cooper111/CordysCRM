---
type: domain-base
sub-type: repository-index
module: customer
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/customer/mapper/ + resources/mapper/customer/
owner: cordys-crm
verify-required: true
verify-note: 表名/字段/索引 易变，核对 Flyway + XML
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/domain-customer.md
  - knowledge/applications/backend/domain/base/repository-core.md
---

# 客户模块 存储索引（repository-customer）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀 | XML 位置 |
|---|---|---|---|
| **ExtCustomerMapper** | 客户主档 CRUD/分页/列表 JOIN 扩展字段 | crm_customer | ExtCustomerMapper.xml |
| **ExtCustomerFieldMapper** | 客户扩展字段行 | crm_customer_field | ExtCustomerFieldMapper.xml |
| **ExtCustomerFieldBlobMapper** | 客户扩展 BLOB | crm_customer_field_blob | ExtCustomerFieldBlobMapper.xml |
| **ExtCustomerContactMapper** | 联系人 CRUD | crm_customer_contact | ExtCustomerContactMapper.xml |
| **ExtCustomerPoolMapper** | 客户池 CRUD | crm_customer_pool | ExtCustomerPoolMapper.xml |
| **ExtCustomerCapacityMapper** | 客户容量 | crm_customer_capacity | ExtCustomerCapacityMapper.xml |
| **ExtCustomerOwnerHistoryMapper** | 负责人历史 | crm_customer_owner_history | ExtCustomerOwnerHistoryMapper.xml |
| **ExtCustomerCollaborationMapper** | 协作人 | crm_customer_collaboration | ExtCustomerCollaborationMapper.xml |
| **ExtCustomerMergeSnapshotMapper** | 合并快照 | crm_customer_merge_snapshot（表名核对） | ExtCustomerMergeSnapshotMapper.xml |
| **ExtCustomerTransformSnapshotMapper** | 转化/反转化快照 | crm_customer_transform_snapshot（核对） | ExtCustomerTransformSnapshotMapper.xml |
| **ExtCustomerDuplicateMapper**（如有） | 去重候选对 | crm_customer_duplicate（核对） | ExtCustomerDuplicateMapper.xml |

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_crm_customer_tables.sql`（具体日期不同，**不允许修改已运行脚本** R5-B）

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 客户分页 SQL / JOIN 扩展字段 | ExtCustomerMapper.xml |
| 客户联系人 一主多从 批量保存 | ExtCustomerContactMapper 的 batchInsert / batchUpdate |
| 客户合并 / 反转化 数据还原 | CustomerMergeSnapshot / TransformSnapshot 表 |
| 客户协作人查询（谁能看客户） | ExtCustomerCollaborationMapper.xml |
| 客户去重（按名称/手机号） | ExtCustomerMapper.xml 的 selectDuplicateCandidates 或 CustomerDuplicate 表 |
