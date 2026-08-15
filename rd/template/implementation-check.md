# RD 模板：实现校验（implementation-check.md）

> 路径：复制到 `rd/requirements/{requirementId}/implementation-check.md`
> 目标：对比 requirement.md 的"需求点清单"和"实际代码改动"，确保没有遗漏、没有多做、没有做错。
> 本质：AI 先做一次自测+CR，人工聚焦漏项和架构问题。

---

## 元信息

| 字段 | 内容 |
|---|---|
| Requirement ID | `{requirementId}` |
| 校验日期 | `YYYY-MM-DD` |
| 校验范围 | {应用} |
| 代码分支 / Commit | `feature/xxx` @ `{commit hash 短}` |
| 基于需求 | requirement.md |

---

## 一、需求点覆盖度校验（最重要的表格，必须逐项核对）

| ID | 需求点 | 需求文件位置 | 代码实现位置（必须写具体路径+函数名） | 核对结果 ✅/⚠️/❌ | 备注/证据 |
|---|---|---|---|---|---|
| R1 | {需求点1} | requirement.md R1 | `{Service}.{method}()` L20-L80 | ✅ | 单测覆盖 + 手动验证通过 |
| R2 | {需求点2} | R2 | | ⚠️ | 已实现，需人工验证边界条件（N=0、N=临界值） |
| R3 | {需求点3} | R3 | | ❌ | 遗漏！代码里未找到对应实现，需补 |
| | | | | | |

**覆盖率统计**：
- 总需求点数：N
- ✅ 已完成正确实现：X
- ⚠️ 实现但需重点人工确认：Y
- ❌ 遗漏 / 错误：Z
- 覆盖率 = X / N（目标 100%，Z=0 才能上线）

---

## 二、代码质量校验（对照知识库约束）

| 校验项 | 标准/知识入口 | 核对结果 ✅/❌ | 备注 |
|---|---|---|---|
| 功能权限注解 | @CsPermission 齐全 | ✅ | 对照 permission.json 节点正确 |
| 操作日志注解 | 所有写操作 @OperationLog | ✅ | SpEL resourceId 绑定正确 |
| 统一响应 ResultHolder | tech-constraints.md 后端铁律 2 | ✅ | 未发现裸返回对象 |
| 异常处理：用 GenericException | tech-constraints.md 后端铁律 3 | ⚠️ | 1 处 catch(Exception) 后直接吞异常，需补日志+重抛 |
| BaseMapper + LambdaQuery | tech-constraints.md 后端铁律 1 | ✅ | 未发现手写原生 SQL |
| 数据权限：过 DataScopeService | business-rules.md 1.3 | ✅ | 查询均已拼接范围条件 |
| （前端）API 调用：用 @lib/shared | lib-shared INDEX | ✅ | 未发现 new axios |
| （前端）按钮权限：v-permission | frontend-web tech | ✅ | |

---

## 三、测试覆盖校验

| 类别 | 要求 | 核对结果 | 证据（测试类名/覆盖率） |
|---|---|---|---|
| 单元测试 | 新增/修改逻辑 ≥80% 行覆盖 | ✅ | ClueExportServiceTest：89% |
| 边界用例 | 空/最小/最大/异常 | ✅ | 含 0 行、1 行、临界值、权限不足等 12 个用例 |
| 集成测试 | 至少 1 条端到端（Testcontainers MySQL） | ✅ | 完整"创建线索→导出→校验内容"链路 |
| 手动验证 | P0 需求点人工过一遍 | ⚠️ | 10W 导出尚未跑，建议压测环境跑一遍 |

---

## 四、风险回归检查（对照 analysis.md 风险表）

| analysis 中风险 | 当前状态 | 已落实的应对措施 |
|---|---|---|
| DB 查询压力 | ✅ 已缓解 | 每页 2000 + sleep 10ms + 单人并发限流（Redis 锁） |
| 导出列兼容 | ✅ 已验证 | 新旧导出 20 个列顺序、格式完全一致 |
| 移动端拦截 | ✅ 已实现 | 超过 5000 行提示 PC 端操作 |

---

## 五、漏项 & 风险清单（必须修复后才能上线）

### ❌ 阻断项（不修复不上线）

1. **R3 需求点遗漏**：{具体描述} → {指定某人/AI 补做}

### ⚠️ 非阻断但上线前建议处理

1. {例如} 1 处吞异常：建议补 log.error + 重新抛出 GenericException
2. {例如} 10W 导出压测：建议在预发环境跑一次，记录真实耗时与内存

---

## 六、校验结论与进入 CR 的签字

- 校验人（AI）：已完成上述全部核对
- 阻断项数：{Z，目标 0}
- 覆盖率：{X/N}%（目标 100%）
- 结论：
  - ✅ **PASS**：可进入人工 Code Review
  - 🔄 **NEED-FIX**：先处理完阻断项，再重新 validate
  - ❌ **FAIL**：偏差过大，回到 requirement 阶段重做
