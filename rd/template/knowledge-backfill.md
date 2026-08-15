# RD 模板：知识回补（knowledge-backfill.md）

> 路径：复制到 `rd/requirements/{requirementId}/knowledge-backfill.md`
> 目标：把本需求过程中沉淀的稳定经验、规则、踩坑，回流进知识库体系。
> 没有回补的 RD，价值只在当次；有回补的 RD，是团队复利资产。

---

## 元信息

| 字段 | 内容 |
|---|---|
| Requirement ID | `{requirementId}` |
| 回补日期 | `YYYY-MM-DD` |
| 需求类型 | 新功能 / Bug 修复 / 优化重构 / 配置变更 |
| 本需求关键产物 | analysis.md / requirement.md / implementation-check.md |
| 线上运行验证（如已上线） | ✅ 已上线稳定 N 天 / ⚠️ 灰度中 / ❌ 尚未发布 |

---

## 一、本需求中沉淀的稳定知识清单（候选 → 正式）

> 稳定性判断：本知识在未来的 3+ 个类似需求中依然适用。
> 每条都必须附：证据代码/文档路径。

| # | 知识标题 | 类型（flow/rule/state/tech/code-snippet） | 可信度 H/M/L | 建议入库路径 | 证据链（3 条以内） |
|---|---|---|---|---|---|
| 1 | {例}线索导出性能优化模式：分页查+流式写+限流+异步任务 | code-snippet | H（10W 行导出压测通过） | knowledge/template/code-snippet.md 的模式可升级为 applications/backend/domain/product/clue.md 章节 | ① ClueExportService.java L20-L120 ② 单测 ClueExportPerformanceTest ③ 压测报告 P95=1.8s |
| 2 | {例}导出列兼容检查清单 | tech | M（UAT 对齐一次通过） | knowledge/applications/backend/tech/excel-io.md 增补"兼容检查"章节 | ① 分析文档 R2 ② implementation-check.md R2 行 |
| 3 | | | | | |

**入库操作**（需求负责人执行或委托）：

- [ ] 将上表第 1 条整理写入 `knowledge/candidate/20260812-clue-export-perf-pattern.md`，填写 owner reviewer（参考 candidate/_template.md）
- [ ] 第 2 条写入 tech 对应文件的增补 PR
- [ ] 其余条目按建议路径落地

---

## 二、本需求沉淀的个人经验（→ personal/）

> 这部分不用 owner review，写入个人目录即可，下次遇到类似问题快速检索。

| # | 经验/踩坑标题 | 作者 | 写入个人目录路径建议 | 1-2 句话摘要 |
|---|---|---|---|---|
| 1 | {例}EasyExcel SXSSF 临时文件目录没权限导致导出失败 | @{个人名} | personal/{name}/pitfall-excel-sxssf-tmpdir.md | SXSSF 会写临时文件到 `java.io.tmpdir`，容器默认 /tmp，需确保 cordys 用户有写权限，或手动指定临时目录 |
| 2 | | | | |

---

## 三、本需求中更正/更新的已有知识

> 如果本需求发现已有正式知识是错的，**不要直接改正式知识**，改法如下：

| # | 发现问题的知识文件 | 问题描述（旧版错误内容） | 正确内容简述 | 处理动作 |
|---|---|---|---|---|
| 1 | （例）knowledge/applications/backend/tech/excel-io.md V1.0 | 旧文档说"FastExcel 默认自动流式"，实际默认是内存模式 | 必须手动开启流式写参数并传 SXSSF 参数 | ① 写入 candidate/ 标注"知识库待更新" ② owner review 后改正式文件，标记 deprecated 旧描述 |
| 2 | | | | |

---

## 四、本需求不建议入库的内容 & 原因

> 不是所有东西都要入库，"什么不入库"同样重要。

| 内容 | 不入库原因（选一个或多个） |
|---|---|
| {例}本次导出时某个字段的临时 SQL 拼接方式 | 仅本次需求适用；下次需求字段就变了 |
| {例}某个测试同学当时测的是 Chrome 125 版本 | 时效性太强，无长期价值 |
| {例}本地调试时端口冲突了，改了个临时配置 | 个人环境问题，非团队通用经验 |

---

## 五、签字确认（需求正式关闭前必须完成）

- [ ] "稳定知识清单"每一条都有明确的代码/文档证据
- [ ] 至少 1 条高可信度内容已写入 candidate/（本需求无任何沉淀可标 N/A，但建议至少有 1 条）
- [ ] 个人经验（如有）已写入对应个人目录
- [ ] 发现的知识库问题已走 candidate 流程（无则标 N/A）
- [ ] 回补人签字：@{user} + AI
- [ ] 知识 owner 抽查签字（可选，但建议每个季度抽查 >30%）：@{owner}
