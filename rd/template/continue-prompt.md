# RD 接续文件（continue-prompt.md）

> 路径：复制到 `rd/requirements/{requirementId}/continue-prompt.md`
> ⚠️ **最重要的文件**：换会话、换人、换 AI 产品时，先读本文件即可恢复全部上下文，避免结论漂移。
> 本文件必须**每次进度推进后手动更新**。

---

## 📌 Requirement ID

`{requirementId}`（例：20260811-clue-export-optimize）

## 📌 当前进度位置

```
verify-prd → clarify → analyze → decompose → verify-requirement
                                                  ↓
                                            [开发阶段 🟢 你在这里]
                                                  ↓
                                         apply → validate → code-review
                                                  ↓
                                         release-plan → 发布 → 知识回补
```

当前所处阶段：**{如：开发阶段 80%，导出主逻辑完成，任务中心还差失败状态写入}**
下一步动作：**{如：补写 R3（导出失败重试）+ 补单测 + 进入 validate}**

---

## ✅ 已完成清单（进度快照，不用再读旧文件）

| 阶段 | 完成日期 | 关键结论 | 产物文件 |
|---|---|---|---|
| clarify | YYYY-MM-DD | {核心澄清结论 1-3 条，不要展开} | clarification.md |
| analyze | YYYY-MM-DD | {目标+非目标+核心变更点 3 条摘要} | analysis.md |
| decompose | YYYY-MM-DD | 拆成 2 个应用：backend(clue) + frontend-web(clue) | decomposition.yaml |
| verify-requirement | YYYY-MM-DD | 拆解无遗漏，契约对齐 | （口头确认，无额外产物） |
| apply（开发） | 进行中 | ClueExportService 分页+流式重写完成 ✅<br>Controller options 参数已加 ✅<br>导出任务失败状态写入 ❌ 未完成 | 代码分支 feature/xxx |

---

## 🎯 下一步 TODO（按优先级排序）

| # | 任务 | 执行者 | 依赖 | 完成条件 |
|---|---|---|---|---|
| 1 | 补：导出任务失败状态写入失败记录 | AI / 开发者 | 无 | 在 ExportTaskService 中写入失败原因 |
| 2 | 补：导出失败场景单测 | AI / 开发者 | 任务1 | 新增 3 个异常 case，全部 pass |
| 3 | 生成 implementation-check.md | AI | 任务1+2 | 阻断项 =0 |
| 4 | 提交人工 CR + llm-code-review workflow | 开发者 | 任务3 | CR 无阻断意见 |

---

## ⚠️ 关键决策与已确认事项（避免再问同样的问题）

| 决策/事项 | 结论 | 确认来源 | 不要再讨论 |
|---|---|---|---|
| 导出列兼容性 | 必须与旧版列顺序 100% 一致 | @{产品经理} 确认 ✅ | ⛔ 不要再提"要不要改列顺序" |
| 权限模型 | 复用 CLUE:EXPORT 节点，不新增权限节点 | analysis.md 非目标章节 ✅ | ⛔ |
| 移动端 | 仅加超限拦截提示，不重写导出 | decomposition.yaml 应用块3 ✅ | ⛔ |
| 分页策略 | 每页 2000 行，页间 sleep 10ms | requirement.md 设计决策 ✅ | ⛔ 不要再纠结"500/5000 哪个更好" |

---

## 🔗 关键上下文入口（接续时优先读取的文件/代码）

| 类别 | 路径 | 说明 |
|---|---|---|
| RD 产物 | rd/requirements/{id}/clarification.md | 已确认澄清结论 |
| RD 产物 | rd/requirements/{id}/analysis.md | 背景/目标/需求点清单源头 |
| RD 产物 | rd/requirements/{id}/requirement.md | 开发任务书（需求点 R1~R5） |
| RD 产物 | rd/requirements/{id}/implementation-check.md | 已核对部分可跳过 |
| 知识库 | knowledge/ROUTING.md + main/glossary.md | 术语与路由（已读） |
| 知识库 | knowledge/applications/backend/domain/product/clue.md | 线索主干能力 |
| 知识库 | knowledge/applications/backend/tech/excel-io.md | 导出规范 |
| 代码（改） | backend/crm/src/main/java/.../ClueExportService.java | 已完成 80%，继续补失败写入 |
| 代码（改） | backend/crm/src/main/java/.../system/service/ExportTaskService.java | 写入任务状态的下游类（已读） |

---

## 🚫 踩过的坑（接续时不要再踩）

| 坑位 | 发生了什么 | 正确做法 |
|---|---|---|
| 坑 1 | 导出时先查全量再分页，结果 10W 行 firstStep 就 OOM | 用 PageHelper.offsetPage + 流式写，while(hasNext) 循环 |
| 坑 2 | 操作日志 SpEL 写成 `#req.id`，实际参数名是 `exportReq` | 对照方法形参名，写 `#exportReq.xxx` |

---

## 🧭 接续操作指引（给接手的 AI/开发者）

> 如果是 AI 接手：
> 1. 先读本文件全部内容（1 分钟掌握上下文）
> 2. 按上方"下一步 TODO"顺序做
> 3. 做完后更新本文件"已完成清单" + "下一步 TODO" + 当前进度位置
> 4. 完成 implementation-check.md 后推进到 code-review

> 如果是人工接手：
> 1. 先读本文件 + analysis.md 核心变更点
> 2. 检查"关键决策"是否仍认同（不认同立刻讨论，别做一半返工）
> 3. 检查 TODO 清单，确认工作量与排期
