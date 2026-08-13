---
type: template
template-for: rule
status: official
updated: 2026-08-11
---

# 知识写作模板：业务规则（rule）

> 适用于单个具体规则的详细说明（非通用规则集合）。
> 例如："线索自动回收入线索池的计算规则"、"赢单审批流匹配条件"、"数据权限范围判断规则"。

---

## 复制以下内容到新文件：

```markdown
---
type: rule
scope: main                      # main 全局 / applications/{app}
rule-id: R-{模块缩写}-{编号}     # 例如 R-CLUE-001
rule-name: {规则中文名，例如：线索超时回收规则}
status: official
evidence: {规则判断所在的 Service/Aspect/Handler 代码入口}
owner: @{负责人}
verify-required: true
verify-note: 判断条件中的阈值、天数、开关配置请核对 {配置文件}
updated: YYYY-MM-DD
related:
  - knowledge/main/business-rules.md#{章节}
---

# 规则：{规则中文名}

> 规则编号：`{rule-id}`
> 一句话说明：{这条规则是判断什么的}

---

## 一、规则判断逻辑（伪代码，方便 AI 直接理解）

```
boolean judge({输入参数}) {
  // 前置跳过条件
  if ({跳过条件，例如：管理员角色 || 已强制关闭回收}) return NO_ACTION;

  // 核心判断（按优先级）
  if ({条件1}) return {结果A};
  else if ({条件2}) return {结果B};
  else return {默认结果};
}
```

---

## 二、自然语言逐条说明

| 步骤 | 判断项 | 命中时的结论 | 未命中时的下一步 |
|---|---|---|---|
| 1 | 规则开关是否开启 | 不执行 | （配置入口：{配置 key}） |
| 2 | | | |
| 3 | | | |

---

## 三、配置项/阈值清单（必填，这些值不能写死在知识里）

| 配置项名 | 语义 | 默认值 | 配置位置（必须写具体路径） |
|---|---|---|---|
| {配置key} | {说明} | N 天 | `installer/conf/cordys-crm.properties` 或 系统设置页面 |

---

## 四、异常/边缘情况

| 边缘场景 | 处理方式 |
|---|---|
| 配置值非法（负数/超大） | 走默认值 + 告警 |
| 多个规则冲突时 | 优先级：{本规则和其他规则的优先级顺序描述} |
| 历史数据没有某个字段 | 兼容处理：{怎么判断新老数据} |

---

## 五、代码入口速查

| 阶段 | 位置（类/方法路径） | 典型函数名 |
|---|---|---|
| 规则判断主入口 | `{Service 完整类名}` | `judgeXxx / checkXxx / computeXxx` |
| 切面拦截（如有权限/审批规则） | `{Aspect 类名}` | |
| 调度触发（定时跑的规则） | `{Job 类名}` | |
| 规则生效后副作用处理 | `{Handler/Listener 类名}` | 发通知/写日志/改状态 |
