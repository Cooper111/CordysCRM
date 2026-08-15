# RD 模板：需求澄清（clarification.md）

> 路径：复制到 `rd/requirements/{requirementId}/clarification.md`
> 目标：结合知识库，对不确认点主动提问，确保理解正确。

---

## 元信息

| 字段 | 内容 |
|---|---|
| Requirement ID | `{requirementId}` |
| 创建日期 | `YYYY-MM-DD` |
| 需求提出人 | @{user} |
| 澄清人（AI + 人工） | @{user} |
| 已读知识库 | 列出读取的 knowledge/ 文件：ROUTING.md、main/xxx.md、applications/xxx/INDEX.md |

---

## 一、原始需求摘录（原文粘贴，便于回溯）

> 复制用户/PRD 的原始描述，不要二次加工。

```
{贴原文}
```

---

## 二、AI 主动澄清清单（AI 必须先读知识库，再提问题）

> 说明：AI 结合 `knowledge/ROUTING.md` + `main/glossary.md` + 对应 applications/ 知识后，整理出不确认点。
> 规则：每个问题必须说明"为什么要问这个问题"（让提需求的人理解背景，不是空问）。

| # | 问题（问用户） | 为什么问这个问题？（AI 说明） | 关联知识/代码路径 | 用户答复 |
|---|---|---|---|---|
| 1 | 例如：这个导出功能涉及"客户协作人"的数据范围吗？ | main/business-rules.md 中"数据权限 DataScopeService"有 SELF / DEPARTMENT / CUSTOMER_COLLABORATION 多种范围，不同范围 SQL 查询不同，不确认容易越权或漏数 | knowledge/main/business-rules.md#1-3 数据权限 | （用户填写：要/不要/仅主负责人） |
| 2 | | | | |
| 3 | | | | |

---

## 三、澄清结论与假设清单（所有答复+AI 做出的合理假设）

> 澄清完成后必须整理成一张表，后续 analysis/requirement 基于此表。

| 项 | 澄清结论 / AI 假设 | 确认状态 ✅/⚠️ |
|---|---|---|
| 1. 数据权限范围 | 仅主负责人 + 协作人可见 | ✅ 用户答复 |
| 2. 兼容导出旧格式 | 假设为：是，保持列顺序与历史一致（如有冲突需再确认） | ⚠️ AI 假设，未获用户答复，若错误请指出 |
| | | |

---

## 四、进入 analyze 阶段的确认

- [ ] 已读完 ROUTING.md + 至少 1 份 main/ + 至少 1 份 applications/
- [ ] 澄清清单所有 ✅ 问题已获得答复
- [ ] ⚠️ 假设项不超过 2 个，且均标注了可回退方案
- [ ] 签字确认可进入 analysis 阶段：@{user}
