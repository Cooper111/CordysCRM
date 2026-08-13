---
type: template
template-for: domain (product / solution)
status: official
updated: 2026-08-11
---

# 知识写作模板：领域能力（domain / product / solution）

> 新建 applications/{app}/domain/product/{module}.md 或 solution/{xxx}.md 时复制本模板。

---

## 复制以下内容到新文件：

```markdown
---
type: domain-product             # domain-product（产品能力）/ domain-solution（方案差异）
app: backend                     # 所属应用
module: clue                     # 业务模块（clue/customer/opportunity/contract/order/...）
status: official
evidence: 代码路径（多个 Controller/Service/Domain）
owner: @{负责人}
verify-required: true            # 业务逻辑大概率要回代码核对
verify-note: 接口签名、枚举值、字段名必须核对当前代码
updated: YYYY-MM-DD
related:
  - knowledge/main/glossary.md#{术语锚点}
  - knowledge/applications/backend/domain/base/api-{module}.md
---

# {模块名} · 产品能力（或：{某业务身份} · 差异化方案）

---

## 一、能力概述

一句话总结这个模块解决什么问题。
主干流程：`输入 → 处理步骤1 → 处理步骤2 → 输出`

---

## 二、核心流程（ASCII 流程图）

```
{状态A} ──{动作1}──▶ {状态B} ──{动作2}──▶ {终态C}
   │                                        ▲
   └────{异常分支，例如：超时回收}──────────┘
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑 | 涉及代码入口（必须写） |
|---|---|---|---|
| 动作1 | {什么条件触发} | {逻辑要点} | `cn.cordys.crm.{module}.service.{Service}.{method}()` |
| 动作2 | {条件} | {逻辑} | {路径} |

---

## 三、核心业务规则（写判断条件，不要写实现细节）

| 规则 ID | 规则描述 | 例外情况 |
|---|---|---|
| R1 | 例如：线索转客户必须先验证客户企业名称不为空 | 管理员强制转化例外 |
| R2 | | |

---

## 四、状态/枚举（仅语义，实际值回代码核对）

| 语义 | 枚举名（**仅定位用，实际值回代码核对**） | 代码入口 |
|---|---|---|
| {语义描述} | {枚举值名} | {路径}/{Enum}.java 或 {enums/*Enum}.ts |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约（接口/消息/DTO） |
|---|---|---|
| {模块X} | 本模块 → X | 调用 xxx API，传 YYY DTO |
| {模块Y} | Y → 本模块（回调） | 订阅 Topic: {topic名} |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| 重复提交/幂等 | {怎么处理：唯一键/去重表/状态判断} | {文件} |
| 并发冲突 | {乐观锁/悲观锁/串行队列} | {文件} |
| 兼容历史数据 | {怎么判断新老数据，怎么兼容} | {文件} |
