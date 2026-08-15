---
type: domain-product
app: backend
module: customer
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/customer 下 Controller/Service/Domain/Mapper（含 Clue→Customer 转化、联系人、协作、公海、容量、合并、关系、反转化等能力）
owner: cordys-crm
verify-required: true
verify-note: 接口签名/DTO字段/权限节点需回代码核对最新
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#客户/联系人/公海/客户协作
  - knowledge/main/core-process.md#3-客户生命周期
  - knowledge/main/business-rules.md#1-权限模型 + #2-容量 + #3-公海
  - knowledge/main/state-defs.md#二客户状态
  - knowledge/applications/backend/domain/base/api-customer.md
  - knowledge/applications/backend/domain/base/domain-customer.md
  - knowledge/applications/backend/domain/base/repository-customer.md
---

# 客户模块 · 产品能力

---

## 一、能力概述

客户模块管理企业**客户主数据**及其生态：客户档案 + 联系人 + 协作人 + 客户关系 + 公海 + 容量 + 客户合并/去重/移交 + 线索转客户/反转化。客户是商机、合同、订单等**全部销售后续动作的根节点**（资源的 owner 链条）。

主干流程：
```
线索转客户/手工录入/导入 → 建立客户档案 + 主联系人 → 协作人分配/关系绑定
     ↓ 跟进/商机/合同  或  ↓ 未跟进超期
 [活跃客户]          [入客户公海] ←──── 容量释放/退回/回收 ───┘
     ↓
  客户合并 / 移交 / 反转化回线索
```

---

## 二、核心流程（ASCII 流程图）

```
录入入口 3 种：
  ┌──────────────────────────────────────────────────┐
  │ ① 线索转客户 / ② 手工新建 / ③ 批量导入           │
  └──────────────────────┬───────────────────────────┘
                         ▼
                  [客户创建校验链]
    联系人唯一性检查 → 企业名查重 → 容量检查（分配给负责人时）
                         │
                         ▼
              Customer（客户表） + CustomerContact（联系人表）
              + CustomerField（自定义字段） + CustomerOwner（负责人）
              + CustomerCollaboration（协作人，可选）
                         │
          ┌──────────────┼───────────────┐
          ▼              ▼               ▼
   【活跃路径】     【协作/关系】       【公海回收路径】
  客户跟进 + 商机 + 合同 + 订单   协作人增删改查     未跟进 N 天/容量超限/手动退回
  (cross-module)              客户关系链(关联公司)      ↓
          │                                              │
          ▼                                              ▼
  客户合并 / 客户移交                        CustomerPoolService.recycle
          │                                              │
          ▼                                              ▼
  合并：保留一条主客户，              客户入公海后 → 其他销售领取 → 回到活跃路径
  把关联资源（商机/合同/联系人/跟进）
  迁移到主客户 ID
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口（定位类+方法） |
|---|---|---|---|
| 线索转客户 | 用户在线索页点击"转客户" | 前置校验→写 Customer+Contact+Field→线索状态改"已转化"→写操作日志；失败整体回滚（跨模块事务） | `CustomerService.transformFromClue()` 调用自 ClueService |
| 客户新建 | 手动在客户列表新增 | 字段校验→联系人查重（手机号/邮箱）→写 Customer + Contact | `CustomerService.add()` |
| 客户导入 | 客户 Excel 导入 | 异步 Import 任务→校验（重复、字段权限）→批量写入；失败下载错误文件 | `CustomerExportService`（导出）+ system/ImportService（导入） |
| 客户合并 | 两个客户信息高度重合点击"合并" | 选主客户→把从客户的：联系人 / 协作 / 商机 / 合同 / 订单 / 跟进计划 / 跟进记录 / 操作日志 → **逐表迁移 resourceId 到主客户** → 从客户逻辑删除 | `CustomerService.mergeCustomer(CustomerMergeRequest)` |
| 客户移交 | 销售离职或客户归属调整 | 校验容量（目标用户容量+N <= 上限）→写 CustomerOwner 表新纪录→写操作日志（移交原因） | `CustomerService.transfer()`（批量 batchTransfer） |
| 客户公海回收 | 客户未跟进 N 天（CustomerPoolRecycleRule）+ 定时任务触发 | CustomerPoolRecycleListener → 批量查询满足条件客户→CustomerPoolService.recycle→写 CustomerPool→清空 owner→保留历史 | `CustomerPoolRecycleListener` Quartz 监听器 + `CustomerPoolService.recycleBatch()` |
| 客户协作人 | 在客户详情→协作人 Tab 新增/删除 | CustomerCollaboration 记录协作人 ID + 协作类型（只读/编辑）→ 写操作日志→数据权限 DataScopeService 识别 CUSTOMER_COLLABORATION | `CustomerCollaborationService`（save/remove）+ business-rules.md#1-3 |
| 客户反转化（还原线索） | 误转化或需要回到线索池跟进 | 校验当前客户尚未关联商机/合同（若有关联需先解除）→ 还原一条对应 Clue → Customer 标记逻辑删除或状态"已还原" | `CustomerService.reTransformToClue()` → ClueService 接收 |

---

## 三、核心业务规则（判断条件）

| 规则 ID | 规则描述（if/then） | 例外 |
|---|---|---|
| R1 | **联系人去重主规则**：if (同组织内联系人手机号+邮箱任一完全重复 AND 不是同客户编辑) then 拒绝新建 | 管理员可强制通过（但记录风险操作日志）；可在模块字段中关闭唯一性开关 |
| R2 | **客户容量**：if (目标销售客户数 >= CustomerCapacity.limit) then 拒绝分配/领取/移交 | 超级管理员分配忽略容量，但记录超容量标识 |
| R3 | **合并不可逆**：if (从客户 ID 已合并到主客户) then 不能撤销（合并动作写操作日志 snapshot 提供审计，不提供还原按钮） | 备份还原：从 DB 操作日志 + 快照手工还原 |
| R4 | **反转化前置条件**：if (客户下有关联商机(非赢非输) / 合同(未完结) / 开票中) then 拒绝反转化回线索 | 管理员可强制反转化，但必须先解除关联（前端强制弹确认 + 风险提示） |
| R5 | **协作人权限**：if (协作人.type = 只读) then 禁止 UPDATE / DELETE 操作，仅允许 SELECT + 跟进记录评论 | 协作人.type = 编辑 → 同 owner 的大部分写权限（不含"移交客户"、"删除"等关键动作） |
| R6 | **客户公海隐藏字段**：同线索池规则（CluePool.hiddenFields），客户公海 CustomerPool.hiddenFields 对领取前的销售隐藏指定字段 | 公海管理员查看不受限制 |
| R7 | **关系图循环**：if (新增客户关系 A→B 后会形成 循环(A→B→A)) then 拒绝新增或提示循环 | 不做 DFS 强制阻断时，关系图展示可能异常，前端需容错展示 |
| R8 | **移交+审批联动**：if (移交时组织配置"客户移交需审批"开关开) then 先提交审批流 → 审批通过 → 执行移交写表 | 审批流由 ApprovalResourceHandler 扩展 |

---

## 四、状态/枚举（语义定位，实际值核对代码）

| 语义 | 枚举/类名 定位入口 | 代码路径 |
|---|---|---|
| 客户状态（潜在/合格/活跃/流失/已回收...） | `cn.cordys.crm.customer.constants.CustomerResultCode` + `Customer.status` 字段映射 | crm.customer.constants/ |
| 客户协作类型（只读/可编辑/仅看跟进） | `cn.cordys.crm.customer.constants.CustomerCollaborationType` | crm.customer.constants.CustomerCollaborationType |
| 客户关系类型（上下游/母子/合作/竞争...） | `cn.cordys.crm.customer.domain.CustomerRelation.type` 字段 + 字典 DictModule.CUSTOMER_RELATION_TYPE | crm.customer.domain.CustomerRelation |
| 客户表单 key | `FormKey.CUSTOMER / CUSTOMER_POOL / CUSTOMER_CONTACT` | common.constants.FormKey |
| 公海回收规则 / 领取规则 | `CustomerPoolRecycleRule / CustomerPoolPickRule`（同 clue 模式） | crm.customer.domain.* |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **clue** | customer ↔ clue | 双向转化：transformFromClue / reTransformToClue；DTO：ClueTransformRequest / BatchReTransitionCustomerRequest |
| **opportunity** | customer → opportunity | 客户下商机：Opportunity.customerId；客户删除/反转化时校验"无进行中商机" |
| **contract / order** | customer → contract/order | 客户合同/订单 Contract.customerId Order.customerId；删除客户前必须先无未完结合同（非作废） |
| **follow（跟进）** | customer → follow | CustomerFollowPlanController/RecordController 调 follow.*Service，resourceType=CUSTOMER + customerId |
| **approval（审批）** | customer ← approval | 客户移交审批 / 工商抬头审批写回 Customer.approvalStatus（若启用） |
| **product/form** | customer → ModuleFieldService | 客户自定义字段（CustomerField/CustomerFieldBlob、CustomerContactField）由 system/ModuleField 管理 |
| **system（通知）** | customer → Notice | 客户分配 / 领取 / 回收时通知相关人；通知事件配置见 NotificationConstants.CUSTOMER_* |
| **system（容量/公海）** | customer → CustomerCapacity + CustomerPool | 容量来自组织配置+角色；公海回收规则 CustomerPoolRecycleRule |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口类 |
|---|---|---|
| **手机号重复但客户不同属（同一个人两个公司）** | 查重时维度是"组织内 + 手机号"；若明确是两个公司关联则允许保留 → 推荐走客户关系关联（CustomerRelation） | CustomerContactService.checkRepeat(ContactUniqueRequest) |
| **客户合并后丢失协作人** | 合并时迁移表清单固定包含 CustomerCollaboration，迁移后从客户协作人 = 主客户协作人 + 从客户协作人（去重） | CustomerService.mergeCustomer → 迁移步骤列表 |
| **导入客户时字段缺失** | 导入流程：CustomerExportService 校验列 → 不匹配列写"忽略列"日志 → 必填项缺失则该条失败（汇总错误行下载） | CustomerExportService + ImportRequest |
| **移交目标销售容量超限** | 先弹"容量不足"提示 → 超管可强制；普通销售需先"释放目标销售的客户（退回公海或转他人）" | CustomerCapacityService.check + CustomerService.transfer precheck |
| **客户下联系人 100+ 查询慢** | 联系人列表走分页（PageHelper），客户详情 Tab 仅展示 TOP N，更多联系人点开分页面板 | CustomerContactService.page + CustomerContactPageRequest |
| **协作人离职未清理** | 协作人不单独校验启用状态：权限 CsPermission 切面在 UPDATE 时会判断用户启用状态 → 若离职则 GET 正常但写动作拒绝；管理员后台清理 | CsPermission 切面 + CustomerCollaboration |
| **反转化后联系人归属** | 反转化时原 CustomerContact 跟随反转化（从客户表删除，但 CustomerContact 保留并关联到新生成的线索对应的 CustomerContact），保证历史可查 | CustomerService.reTransformToClue 的迁移步骤 |
