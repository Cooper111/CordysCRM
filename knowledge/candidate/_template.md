---
type: candidate
status: candidate
confidence: low
source-req-id: TBD
evidence:
  - TBD 代码路径1
  - TBD 代码路径2
  - TBD 需求/聊天记录/测试证据
owner-reviewer: @{TBD}
open-questions:
  - TBD 尚未确认的问题1
  - TBD 尚未确认的问题2
updated: YYYY-MM-DD
---

# 候选知识：{主题中文名}

> ⚠️ 本知识为 **候选（candidate）**，尚未经过 owner review。
> 引用前必须：结合当前代码二次确认，且标注"来自候选知识，可信度待确认"。
> 可信度：`{high / medium / low}`
> 来源需求：rd/requirements/{source-req-id}/

---

## 一、知识内容（套用 template/ 中对应模板，这里仅举通用骨架）

{把经过推理/经验整理出的知识写在这里。
 建议套用对应的知识模板（flow / rule / state / tech / domain-product / code-snippet）。
 不确定的地方标注 [TODO: 待确认]，并登记到上方 open-questions。}

---

## 二、证据链（为什么我们觉得这个是对的）

| 序号 | 证据内容 | 代码/文档/聊天路径 | 可信度贡献 |
|---|---|---|---|
| 1 | | | 单点观察 / 交叉验证 / 线上验证通过 |
| 2 | | | |
| 3 | | | |

---

## 三、待确认项清单（对应 open-questions）

| # | 问题 | 需要谁确认 | 期望完成日期 | 确认结论 |
|---|---|---|---|---|
| 1 | {open-questions[1]} | owner: @xxx | YYYY-MM-DD | （确认后回填：通过/驳回+原因） |
| 2 | | | | |

---

## 四、Review 结论（owner 填写）

- Reviewer：@{owner 签名}
- 日期：YYYY-MM-DD
- 结论：✅ approved（可合入） / ❌ rejected（原因：___） / 🔄 need-more-evidence
- 合入目标路径：knowledge/main/{xxx}.md 或 knowledge/applications/{app}/domain/{xxx}.md
