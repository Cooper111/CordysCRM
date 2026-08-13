# 候选知识暂存区 · 索引（candidate INDEX）

> candidate/ 是知识库的"暂存区"。所有未经过 owner review、未核对当前代码的知识先放这里，再经 review 合入正式区（main/ 或 applications/）。

---

## 一、流转规则

```
personal 个人经验整理
    │  整理：补充证据 + 标注可信度
    ▼
candidate/（本目录）
    │  Review：代码核对 + owner 确认
    ▼
正式知识（knowledge/main/ 或 knowledge/applications/）
    │
    ▼ 代码变化 / 业务规则变化
更新 or deprecated 标记
```

---

## 二、候选知识入区必填字段

每个 candidate 文件必须在 YAML Front Matter 中标注：

| 字段 | 说明 | 示例 |
|---|---|---|
| `type` | 知识类型（和 template/ 对应） | `domain-product` / `rule` / `flow` / `state` / `tech` / `code-snippet` |
| `status` | 固定：`candidate` | `candidate` |
| `confidence` | 可信度：`high` / `medium` / `low` | `medium` |
| `source-req-id` | 来源需求 ID（rd/requirements 目录名） | `20260811-clue-export-optimize` |
| `evidence` | 证据列表（代码路径/聊天记录/测试结果） | 3+ 条证据路径 |
| `owner-reviewer` | 待确认的 owner | `@zhangsan`（负责该模块的同学） |
| `open-questions` | 尚未确认的问题列表（YAML 列表） | `- 状态 E 是否真的有中间状态？` |
| `updated` | 放入 candidate 的日期 | `2026-08-11` |

---

## 三、可信度分级

| 等级 | 含义 | 进入正式区的前置条件 |
|---|---|---|
| **high** 高 | 有 3+ 条独立代码证据交叉验证，或有 1 次线上成功经验 | 仅需 owner 签字确认 |
| **medium** 中 | 有 1-2 条代码证据，且逻辑自洽无矛盾 | owner review + 核对 1 个相关需求的实际代码 |
| **low** 低 | 来自个人推理/单次观察，或有可疑点 | 不建议直接入库，先补充更多证据或在 rd/ 中继续验证 |

---

## 四、候选知识清单

> 所有 candidate/ 下的知识文件登记在此，便于检索和 review。

| 候选文件 | 类型 | 标题 | 可信度 | 来源需求 | 待确认 owner | 入库状态 |
|---|---|---|---|---|---|---|
| `_template.md` | candidate 模板 | （新候选知识复制此模板改名） | — | — | — | `pending` |
| | | | | | | |
| | | | | | | |

**状态字段说明**：
- `pending`：待 review
- `in-review`：owner review 中
- `approved`：确认可合入正式区（等待移文件）
- `rejected`：证据不足或推断错误，不入库（保留记录防止重复工作）
- `merged`：已合入 main/ 或 applications/（本文件可删除或标记 deprecated）

---

## 五、新建候选知识步骤

1. **复制** `candidate/_template.md` 改名为 `{日期}-{缩写主题}.md`（如 `20260811-clue-auto-recycle-threshold.md`）
2. **填写** YAML Front Matter 所有必填字段（见第二条）
3. **正文** 套用 template/ 中对应的知识模板，**不要**把 status 改成 official，保持 candidate
4. **登记** 到上方的候选知识清单表格
5. **通知** owner-reviewer（通过 RD 流程的 knowledge-backfill.md 自动沉淀）
