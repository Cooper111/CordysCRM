---
type: domain-base
sub-type: domain-index
module: customer
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/customer/ 目录
owner: cordys-crm
verify-required: true
verify-note: 字段/枚举值 易变，本文件仅写定位入口
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/customer.md
  - knowledge/applications/backend/domain/base/api-customer.md
---

# 客户模块 领域模型索引（domain-customer）

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 对应表前缀 | 对应 ExtMapper | 字段扩展模式 |
|---|---|---|---|---|
| **Customer** | 客户主实体 | crm_customer | CustomerExtMapper | Y（CustomerField + CustomerFieldBlob） |
| **CustomerField** | 客户扩展字段行存储 | crm_customer_field | CustomerFieldExtMapper | N |
| **CustomerFieldBlob** | 客户扩展 BLOB | crm_customer_field_blob | CustomerFieldBlobExtMapper | N |
| **CustomerContact** | 客户联系人（一个客户多个联系人） | crm_customer_contact | CustomerContactExtMapper | Y?（通常联系人也扩展，核对 CustomerContactField + Blob 是否存在） |
| **CustomerPool** | 客户池主实体 | crm_customer_pool | CustomerPoolExtMapper | N |
| **CustomerCapacity** | 客户容量 | crm_customer_capacity | CustomerCapacityExtMapper | N |
| **CustomerOwnerHistory** | 客户负责人变更历史 | crm_customer_owner_history | CustomerOwnerHistoryExtMapper | N |
| **CustomerCollaboration** | 客户协作人：customerId、userId、collaborationType（只读/可写/主管） | crm_customer_collaboration | CustomerCollaborationExtMapper | N |
| **CustomerMergeSnapshot** | 客户合并快照：被合并客户 id → 目标客户 id、合并时间、操作人、被合并客户 JSON 快照（方便反转化/审计） | crm_customer_merge_snapshot（核对） | CustomerMergeSnapshotExtMapper | N |
| **CustomerDuplicate** | 客户去重候选（按规则命中的重复客户对，待人工确认） | crm_customer_duplicate（如有） | CustomerDuplicateExtMapper | N |
| **CustomerTransformSnapshot** | 线索转客户 / 客户反转化回线索 快照：fromResource（CLUE/CUSTOMER）、toResource、快照 JSON | crm_customer_transform_snapshot（核对） | CustomerTransformSnapshotExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.customer.dto.*`
- export：`cn.cordys.crm.customer.dto.export.*`

---

## 二、定位技巧

| 关键词 | 类 |
|---|---|
| 客户主档/客户详情/客户列表 | Customer |
| 客户 自定义字段 | CustomerField + CustomerFieldBlob |
| 客户联系人（姓名/手机/职位/微信） | CustomerContact |
| 客户公海/客户池 | CustomerPool |
| 客户容量 | CustomerCapacity |
| 客户负责人变更历史 | CustomerOwnerHistory |
| 客户协作人/谁能看客户/协作权限 | CustomerCollaboration |
| 客户合并（A 客户并入 B）/ 合并后回滚 | CustomerMergeSnapshot |
| 客户去重/可能重复的客户对 | CustomerDuplicate（如有） |
| 线索↔客户 转化 / 反转化 快照 | CustomerTransformSnapshot |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **CustomerStatusEnum** | 客户状态（潜在/正式/已流失/已回收…，核对代码） |
| **CustomerLevelEnum** | 客户等级（A/B/C/D，通常字典表不写枚举） |
| **CollaborationTypeEnum** | 协作类型（只读/编辑/主管等） |
