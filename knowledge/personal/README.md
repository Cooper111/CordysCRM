# personal/ · 个人经验目录说明

本目录用于存放**个人**的研发经验、踩坑记录、问题排查笔记、理解草稿。

## 目录规则

1. **子目录按人分**：每个人一个子目录，用英文名/花名，如 `personal/xuanjie/`
2. **个人目录内自由组织**：可以按日期分、按模块分，无强制模板
3. **不作为团队事实引用**：AI 引用 personal/ 中的内容时，必须明确标注"来自个人经验笔记，非团队共识"
4. **可升级流转**：好的个人经验 → 整理后放入 candidate/ → review 后合入正式知识

## 推荐命名

```
knowledge/personal/{你的名字}/
├── pitfall-{模块}-{日期}.md      # 踩坑记录：例 pitfall-clue-20260811.md
├── howto-{动作}-{日期}.md        # 操作指南：例 howto-debug-quartz.md
├── notes-{主题}.md               # 理解草稿：例 notes-approval-flow.md
└── debug-{问题描述}.md           # 问题排查：例 debug-mysql-deadlock.md
```

## 知识流转示例

```
xuanjie 某天排查"线索回收导致的容量负数"
  → 写 personal/xuanjie/pitfall-clue-capacity-negative-20260812.md
     （症状/根因/临时方案/根本方案/影响范围）
  → 方案验证通过 & PR 合入后
  → 整理成 candidate/20260812-clue-capacity-negative.md
     （补证据链 + 标可信度 + 找 clue 模块 owner review）
  → owner 确认后
  → 合入 applications/backend/tech/common-pitfalls.md + 对应 rule
```

.
