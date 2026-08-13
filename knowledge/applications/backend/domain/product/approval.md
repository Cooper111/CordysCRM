---
type: domain-product
app: backend
module: approval
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/approval 下：flow（审批流定义+版本）、instance（实例）、node（节点+链接+条件+审批人）、task（任务+加签）、record（记录+快照+退回）、service（Flow/Instance/Todo/Action/Resource）、handler（资源扩展点）、aspect（HitApprovalAspect）、constants（枚举全集）
owner: cordys-crm
verify-required: true
verify-note: 审批动作枚举/节点条件表达式/多审批人模式/加签类型等易变项必须回代码核对最新
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#审批流/审批节点/加签/抄送
  - knowledge/main/core-process.md#7-审批工作流链路
  - knowledge/main/state-defs.md#九审批工作流状态
  - knowledge/main/business-rules.md#4-审批触发规则（审批命中时机+表单类型）
  - knowledge/applications/backend/domain/base/api-approval.md
  - knowledge/applications/backend/domain/base/domain-approval.md
  - knowledge/applications/backend/domain/base/repository-approval.md
---

# 审批流模块 · 产品能力

---

## 一、能力概述

通用审批工作流引擎，覆盖 Cordys CRM 中所有需要审批的资源（合同/工商抬头/客户移交/线索分配/订单/商机阶段推进/自定义表单数据 等）。核心由三大部分组成：
1. **设计态**：审批流定义 Flow（+多版本 FlowVersion）+ 节点 Node（审批人/条件/链接）
2. **运行态**：审批实例 Instance（对应一次真实提交）+ 节点运行态（当前走到哪）+ 任务 Task（谁待处理）+ 加签任务
3. **动作层**：通过/驳回/加签/减签/退回/撤回/撤销/转办/催办 + WebHook 回调 + 资源处理器（审批通过后更新资源状态）

设计模式：业务对象通过 `@HitApproval` 注解声明"提交动作触发审批"，由 `HitApprovalAspect` AOP 切面统一拦截，命中审批则挂起业务逻辑、生成审批实例；审批通过后由 `ApprovalResourceHandler` 扩展点回调到业务模块更新状态。

主干流程：
```
某业务动作（提交合同审批）→ @HitApproval → HitApprovalAspect 拦截
  → 命中审批流（按 FormType + 组织配置）→ 生成 ApprovalInstance
  → 生成首节点 Task（按节点条件选审批人）
  → 审批人做动作（通过/驳回/加签...）→ 节点流转
  → 末节点通过 → ApprovalResourceHandler 回调 → 业务资源 status=通过，继续原业务逻辑
  → 任一节点驳回 → 资源 status=驳回，业务逻辑终止 + 通知发起人
```

---

## 二、核心流程（ASCII 流程图）

```
 ┌─────────── 业务提交（Controller 写操作方法） ───────────┐
 │ @HitApproval(formType=CONTRACT, action="SUBMIT")        │
 └──────────────────────────────┬──────────────────────────┘
                                ▼
              HitApprovalAspect（AOP 切面）：
               查询该 FormType + 组织 是否有启用审批流
                       没有 → 放行，直接执行业务方法
                       有 → 走下方审批链路 ↓
                                │
                                ▼
                   【创建 ApprovalInstance】
                   关联 resourceId + resourceType + 发起人
                   + ApprovalResourceSnapshot（资源快照防篡改）
                                │
                                ▼
                    【展开首节点】FlowVersion → firstNode
                       - 计算审批人（角色/部门/上级/自选/岗位）
                       - 多审批人模式：串行/并行/会签/或签
                       - 条件节点：按 RuleCondition 表达式计算走哪条分支
                                │
                                ▼
                    【为每个审批人生成 Task】
                        Task 类型=TO_BE_APPROVED（待办）
                        → 站内信 + 邮件 + IM 通知审批人
                                │
               ┌────────────────┴─────────────────┐
               ▼                                  ▼
         审批人通过（PASS）                    审批人驳回（REJECT）
            - nextNode                          - instance.status = REJECTED
            - 若 nextNode = 结束节点              - 通知发起人
              → instance.status = APPROVED        - 不回调 ResourceHandler
              → ApprovalResourceHandler          - 业务资源标驳回状态
                回调：业务资源=通过
            - 否则：生成下一节点 Task
               │
               ▼
   ┌──── 审批人 其他动作 ────┐
   │ 加签 ADD_SIGN：前加/后加 → ApprovalAddSignTask │
   │ 减签 DEL_SIGN：移除某审批人（权限）           │
   │ 退回 RETURN：回到某节点（非首节点）           │
   │ 撤回 REVOKE：发起人撤回未被审批的审批          │
   │ 撤销 ABANDON：发起人撤销整次审批（不可逆）    │
   │ 转办 TRANSFER：把任务转给别人                 │
   │ 催办 URGE：给待办人发二次提醒                 │
   │ 抄送 CC：通知某人看但无审批权（记录）          │
   └──────────────────────────────────────────────┘
                              │
                              ▼
                  【所有动作写 ApprovalRecord】
                  （审批时间/审批人/动作/意见/附件）
                              │
                              ▼
              【可选：WebHook 推送回调】
              组织配置 WebHook → 按 instance.status 回调第三方系统
              （通过/驳回/节点通过 / 每动作）
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 切面拦截提交 | Controller 方法带 @HitApproval 注解，业务发起提交 | 切面先查：该组织该 FormType 是否存在"启用" Flow（最新 FlowVersion）→ 有则走审批，记录快照；无则直接放行 | approval.aspect.HitApprovalAspect（类路径核对） |
| 审批实例 + 首节点 | 切面命中审批 | 写 Instance + ResourceSnapshot（关键）+ 展开首节点 + 计算审批人 → 生成 Task（按多审批人模式） | ApprovalInstanceService.start（+ApprovalFlowService + ApprovalTaskService） |
| 通过 / 驳回 / 加签 等动作 | 审批人点对应按钮 → ApprovalActionRequest | ApprovalActionService：按 action 枚举分发 → 动作前鉴权（用户是审批人 or 管理员）→ 更新 instance + task 状态 → 写 record → 下节点或回调 | ApprovalActionService.execute（ApprovalActionRequest + ApprovalAction 枚举） |
| 条件分支节点 | 进入节点类型=CONDITION 的节点 | 解析节点的 ApprovalNodeCondition（RuleConditionDTO JSON 表达式）→ ConditionFilterUtils 对 resourceSnapshot 求值 → 选中一个 ApprovalNodeLink → 下一节点 | ApprovalNodeLink + ConditionFilterUtils（common.utils） |
| 多审批人模式 | 某节点多人审批 | 按 MultiApproverModeEnum：或签（1 人过即过）/ 会签（全过）/ 串行（按顺序）→ 更新节点状态，决定是否生成下一节点任务 | ApprovalTaskService 任务完成时判断 |
| 审批通过回调资源 | 实例状态=APPROVED | 按 FormType 找对应 ApprovalResourceHandler 实现 → handler.onApproved(resourceId) → 更新业务资源状态（如 contract.approvalStatus=通过） | approval.handler.ApprovalResourceHandler（接口，各业务模块 @Component 实现） |
| 审批定义配置 | 管理后台→审批流设计页 | 审批流 Flow CRUD + FlowVersion 版本管理 + 节点 Node / 链接 Link / 条件 Condition / 审批人 Approver 配置 + 启用/停用（启用=设成当前生效版本） | ApprovalFlowService + ApprovalFlowVersionService |
| 待办/已办/我发起的/抄送 | 工作台 Tab 列表 | ApprovalTodoService.page（按当前用户+状态查 Task）→ 封装成 ApprovalTodoResponse（包含资源信息、可执行动作） | ApprovalTodoService + ApprovalInstanceController.page |

---

## 三、核心业务规则

| 规则 ID | 规则描述（if/then） | 例外 |
|---|---|---|
| R1 | **资源快照不可变**：if (instance 已创建) then ApprovalResourceSnapshot 永不 UPDATE（防止业务方修改资源后审批结论不匹配） | 管理员可刷新快照（需要重新审批，标记 instance 状态=作废，开新 instance） |
| R2 | **审批人计算必须实时**：if (节点配置的"按角色选审批人") → 每次 Task 生成**实时**查询当前用户角色，不能缓存（可能角色昨天刚变） | 节点为"指定人 user_id"类型固定，不重新查 |
| R3 | **撤回仅限发起人且无审批动作**：if (有任何 Task 被操作（PASS/REJECT/任何动作）) then 发起人不允许撤回（只能走驳回或撤销） | 管理员可强制撤回（需记录操作日志风险原因） |
| R4 | **驳回直接到发起人**：if (某节点驳回) → 整次实例=REJECTED，不允许"回到上一个节点重新走"；要走需要发起人重新提交（新 instance） | 如果组织配置"驳回回到当前节点重审"=开 → 驳回后回到当前节点，重新生成该节点 Task |
| R5 | **加签类型**：前加签（加在当前审批人之前，先他审）/ 后加签（之后）/ 并签（和当前人并行）；三种类型分别对应 ApprovalAddSignType（不同枚举），不能混用 | — |
| R6 | **抄送不算在流程内**：抄送 CC 只写记录 + 发通知；不影响 Task/Instance 状态，不要求被抄送人做动作 | — |
| R7 | **审批动作不可逆（除"撤回"）**：一旦选择 PASS / REJECT → 记录 ApprovalRecord，该 Task 结束，无法反悔 | — |
| R8 | **同一资源不允许并发多实例**：if (resourceId + formType + 状态=审批中) 的 Instance 存在 >1 → 拒绝第二次提交（由切面 + 唯一键保证）；避免同一份资源同时被两份审批流通过 | 超管可先作废老实例再提交 |
| R9 | **WebHook 失败重试 3 次**：WebHook 配置的 URL 推送失败 → 间隔 5s/30s/2min 重试 3 次 → 仍失败则记录"推送失败日志"，不影响审批结果 | 管理员后台手动触发重推 |

---

## 四、状态/枚举（定位入口）

| 语义 | 枚举/类名入口 | 代码路径（全部在 approval.constants/ 下，核对代码） |
|---|---|---|
| 审批动作（通过/驳回/加签/减签/退回/撤回/撤销/转办/催办/抄送） | `ApprovalAction` 枚举 | crm.approval.constants.ApprovalAction.java |
| 审批实例状态（草稿/审批中/已通过/已驳回/已撤销/已废弃） | `ApprovalStatus` 或 `ApprovalState`（核对哪个） | constants.ApprovalStatus.java |
| 节点类型（审批/条件/抄送/开始/结束） | `ApprovalNodeTypeEnum` | constants.ApprovalNodeTypeEnum.java |
| 多审批人模式（或签/会签/串行/并行） | `MultiApproverModeEnum` | constants.MultiApproverModeEnum.java |
| 审批人类型（角色/部门/用户/上级/发起人自选/岗位） | `ApproverTypeEnum` | constants.ApproverTypeEnum.java |
| 加签类型（前加/后加/并加） | `ApprovalAddSignType` | constants.ApprovalAddSignType.java |
| 审批表单类型（合同/工商抬头/客户移交/自定义表单...） | `ApprovalFormTypeEnum` | constants.ApprovalFormTypeEnum.java |
| 审批任务类型（普通/加签） | `ApprovalTaskType` | constants.ApprovalTaskType.java |
| 审批人方向（上/下/左/右 退回） | `ApproverDirectionEnum` + ApproverLevelEnum | constants.ApproverDirectionEnum.java |
| 执行时机（提交前/提交后 触发？） | `ExecuteTimingEnum`（@HitApproval 切面用） | constants.ExecuteTimingEnum.java |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **contract / customer / clue / opportunity / order / system** | 业务模块 → approval（注解触发） | 业务方写操作方法加 `@HitApproval(formType = Xxx, action=SUBMIT)`；切面命中后生成审批流实例 |
| **business ← approval（回调）** | approval → 业务模块（ResourceHandler） | 各业务模块实现 `ApprovalResourceHandler` 接口（@Component）→ onApproved / onRejected 两方法，负责把业务资源状态改成 通过 / 驳回 |
| **system（模块字段/字典/通知）** | approval → system | 通知（审批待办/催办/审批通过）走 CommonNoticeSendService；字典；操作日志 + 操作审计 |
| **system（组织配置）** | approval ← system | 审批开关、驳回重审开关、WebHook 配置、审批流表单类型的启用与否均在 OrganizationConfig + 相关 DTO（ApprovalPostConfigDTO） |
| **WebHook（第三方）** | approval → 外部 HTTP | ApprovalPushParam → 推 WebHookConfig.url；失败 3 次重试（见 R9） |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **审批人离职（中途停用）** | 每次 Task 生成时实时校验 User.status=启用；如果已经生成 Task 后用户被停用 → 管理员看到"被停用审批人"标识 → 管理员转办给下一位或强制通过 | ApprovalTaskService.list 过滤 + ApprovalActionService.transfer |
| **条件节点表达式永远为真（死循环）** | 节点深度保护：每实例最大 50 次节点流转 → 超过自动标记实例 = ERROR 终止，通知管理员，不级联驳回 | ApprovalActionService 节点流转计数 |
| **多条件分支都不命中** | 条件节点默认 default 分支 = 结束节点（直接通过）或拒绝（看 FlowVersion 配置）；配置期前端校验必须至少覆盖所有分支 | ApprovalNodeCondition 求值 none-match → default |
| **并发 2 个审批人同时点通过（会签/或签判断）** | Task 动作更新时用乐观锁（task.updateCount）→ 后提交的重新判断：此时该节点其他人是否已全过 → 或签情况下第一人先到直接过，第二个人判定"节点已结束，忽略" | ApprovalTaskService 更新时 version 乐观锁 |
| **资源被删除但审批还在** | 删除资源前切面先查：有无该资源的审批中 Instance → 有则拒绝删除（或选择"级联作废关联审批"）；否则被删的资源审批会导致回调 handler 找不到资源（try/catch 记录错误） | 业务模块 delete 方法前 check + ResourceHandler 容错 |
| **流程版本升级过程中老实例怎么办** | 实例永远绑定其启动时的 FlowVersion（snapshot 时保存 versionId）；升级新版本仅影响后续新实例（老实例继续按老版本走完）→ 避免"老流程跑到一半突然规则变了" | ApprovalInstance.flowVersionId 外键关联 |
