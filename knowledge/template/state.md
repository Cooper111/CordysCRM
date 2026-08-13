---
type: template
template-for: state
status: official
updated: 2026-08-11
---

# 知识写作模板：状态定义（state）

> 新建 knowledge/main/state-defs.md 的子模块说明，或 applications 级状态补充时使用。

---

## 复制以下内容到新文件：

```markdown
---
type: state
scope: main                 # main（全局）/ applications/{app}/{module}
module: {模块名}
status: official
evidence: {前端枚举文件路径 + 后端 Domain 枚举定义位置}
owner: @{负责人}
verify-required: true       # 状态值极易变化，强制要求核对代码
verify-note: 编码前必须回前端枚举 TS + 后端 Domain Java 核对实际枚举名和编码
updated: YYYY-MM-DD
related:
  - knowledge/main/state-defs.md
---

# {模块名} · 状态定义

---

## 一、主状态列表

| 顺序 | 枚举名（定位用） | 中文名 | 语义说明 | 是否终态 |
|---|---|---|---|---|
| 1 | `XXX_1` | | | 否/是 |
| 2 | `XXX_2` | | | 否/是 |

⚠️ **强制核对**：实际枚举字符串值需回到下列位置核对当前代码：
- 前端: `frontend/packages/lib-shared/enums/{module}Enum.ts`
- 后端 Domain: `backend/crm/src/main/java/cn/cordys/crm/{module}/domain/*.java`
- 配置: `backend/crm/src/main/resources/form/field.json` → 对应模块 state 字段配置

---

## 二、状态流转图

```
{状态A} ──{触发条件1 / 接口名}──▶ {状态B}
  │                                      │
  │{触发条件2}                           │{触发条件3}
  ▼                                      ▼
{状态C（终态）} ◀────{触发条件4}────── {状态D}
```

### 流转规则表

| 当前状态 → 目标状态 | 触发条件 | 操作入口（Controller 方法/定时任务） | 必要前置校验 |
|---|---|---|---|
| A → B | {条件} | `{Controller.method}` | |
| B → D | {条件} | | |
| D → C | {条件} | | |

---

## 三、特殊状态（如有）

| 特殊状态 | 语义说明 | 典型使用场景 |
|---|---|---|
| 审批中 PROCESSING | 单据处于审批流运行中 | 期间禁止修改关键字段 |
| 作废 VOID / 归档 ARCHIVED | 终态不可逆转 | 误操作/取消时走作废，正常结束走归档 |

---

## 四、状态与前端展示映射

| 状态 | 展示文案 | 颜色（Naive UI tag type） | 配置位置 |
|---|---|---|---|
| | | | 前端 config/{module}.ts 或 组件内映射 |
