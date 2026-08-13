---
type: domain-base
sub-type: domain-index
module: clue
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/clue/ 目录
owner: cordys-crm
verify-required: true
verify-note: 字段名/字段类型/字段数量 均为易变项（R4 W4），必须以代码为准，本文件只写定位入口
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/clue.md
  - knowledge/applications/backend/domain/base/api-clue.md
  - knowledge/applications/backend/domain/base/repository-clue.md
---

# 线索模块 领域模型索引（domain-clue）

> 本文件只负责"从需求关键词 → 定位到哪个类"。字段/字段关系/具体枚举值 一律回代码查。

---

## 一、核心 Domain 类定位

| Domain 类名 | 职责一句话 | 对应表（表名前缀） | 对应 ExtMapper | 字段扩展？（Y/N + 模式） |
|---|---|---|---|---|
| **Clue** | 线索主实体：主键、title、客户名称、联系人、来源、行业、状态、负责人、nextFollowTime 等核心字段 | crm_clue（具体表名核对代码 @TableName） | ClueExtMapper（或 ClueMapper + XML） | Y（ClueField + ClueFieldBlob，模块字段扩展模式） |
| **ClueField** | 线索扩展字段"元数据行"：每条线索有 N 行（字段 key=value），关联系统模块字段配置 | crm_clue_field | ClueFieldExtMapper | N（这是扩展存储本身） |
| **ClueFieldBlob** | 线索扩展字段 BLOB 存储：超长字段（多行文本/富文本/附件 JSON）存这里，key 与 ClueField 一致 | crm_clue_field_blob | ClueFieldBlobExtMapper | N |
| **CluePool** | 线索池主实体：池名称、池规则（分配规则、回收规则 JSON）、是否默认池、容量配置 | crm_clue_pool | CluePoolExtMapper | N（极少扩展） |
| **CluePoolRecycleRule** | 线索池回收规则独立表（如拆分为独立类）：多少天未跟进自动回池、容量超限回收开关等 | crm_clue_pool_recycle_rule（核对） | CluePoolRecycleRuleExtMapper | N |
| **ClueCapacity** | 线索容量：某用户（userId）的 maxCapacity、usedCapacity、quotaType、quotaFrom、quotaExpireTime | crm_clue_capacity | ClueCapacityExtMapper | N |
| **ClueOwnerHistory** | 线索负责人变更历史：fromUserId / toUserId / assignReason / assignType / operateTime | crm_clue_owner_history | ClueOwnerHistoryExtMapper | N |
| **PoolCluePickRecord** | 池领取记录：userId、clueId、pickTime、pickReason（手动/自动分配） | crm_pool_clue_pick_record | PoolCluePickRecordExtMapper | N |
| **ClueQuarantined** | 线索隔离期（重复防护）：线索 key（手机号/邮箱/客户名 hash） + quarantinedUntil、ownerUserId（隔离归属） | crm_clue_quarantined（核对表名） | ClueQuarantinedExtMapper | N |

### DTO 包位置（只写定位，不展开字段）
- **request**：`cn.cordys.crm.clue.dto.request.*`（ClueCreateRequest / ClueUpdateRequest / ClueAssignRequest / CluePageQueryRequest / CluePoolCreateRequest …）
- **response**：`cn.cordys.crm.clue.dto.response.*`（ClueDetailResponse / CluePageResponse / CluePoolDetailResponse / ClueCapacityResponse …）
- **export DTO**：`cn.cordys.crm.clue.dto.export.*`（ClueExportDTO / CluePoolExportDTO …）

---

## 二、定位技巧（需求关键词 → 找哪个 Domain 类）

| 需求关键词 | 找哪个类 |
|---|---|
| 线索标题/手机号/状态/负责人 | Clue 主实体 |
| 线索 自定义字段 / 扩展字段 / "我在列表里加了个字段" | ClueField + ClueFieldBlob（配合 system.ModuleField 元数据） |
| 线索池 / 分配规则 / 回收规则 / 默认池 / 多个池 | CluePool + CluePoolRecycleRule（若拆） |
| 容量超了 / 领取被拒 / 提升 配额 | ClueCapacity |
| 某线索 换过几个销售 / 负责人变更审计 | ClueOwnerHistory |
| 谁从池里领了线索 / 领过几次 | PoolCluePickRecord |
| 重复防护 / 新建线索冲突 / "被占用隔离" | ClueQuarantined |

---

## 三、枚举类定位入口（只写类名，具体值核对代码）

| 枚举类名 | 说明 | 包路径前缀 |
|---|---|---|
| **ClueStatusEnum** | 线索状态枚举（新建/已分配/跟进中/已转化/已回池/作废…，核对代码） | cn.cordys.crm.clue.enums |
| **ClueAssignTypeEnum** | 分配类型（手动分配/自动分配/销售领取/回收重分配…） | cn.cordys.crm.clue.enums |
| **ClueSourceEnum** | 线索来源枚举（官网注册/市场活动/转介绍/QCC 导入/招标…，字典表更易变，核对是否走 Dictionary 而非枚举） | cn.cordys.crm.clue.enums |

> 注意：线索来源、行业、客户级别 等"业务可配置项"通常不用枚举，走 Dictionary/ModuleField，不要被类名迷惑，核对实际代码用法。
