# RD 模板：应用级需求（requirement.md）

> 路径：复制到 `rd/requirements/{requirementId}/requirement.md`
> 目标：给单个应用/模块的"开发任务书"，是 /rd:apply 和人工编码的直接依据。
> 原则：必须自包含（读完本文件就能写代码，不用再回去翻 PRD）。

---

## 元信息

| 字段 | 内容 |
|---|---|
| Requirement ID | `{requirementId}` |
| 目标应用 | `{backend / frontend-web / frontend-mobile / lib-shared / installer}` |
| 目标模块 | `{clue / customer / ...}` |
| 基于分解 | decomposition.yaml 第几个应用块 |
| 代码分支建议 | `feature/{requirementId}` |

---

## 一、开发目标

{一句话总结这个应用要改什么，对应分析中的哪些变更点。}

---

## 二、必做需求点清单（开发完成后 implementation-check 要逐项核对）

| ID | 需求点（可验证） | 对应 analysis 变更点 | 优先级 P0/P1/P2 | 验收方法 |
|---|---|---|---|---|
| R1 | {具体到可编码的粒度} | analysis.md 变更点 1 | P0 必做 | 单元测试 + UAT |
| R2 | {例如：导出查询必须分页 pageSize=2000，每页 sleep 10ms} | 变更点 1 | P0 | 观察日志分页次数 + 耗时 |
| R3 | {例如：复用现有 CLUE:EXPORT 权限，不新增节点} | 变更点 2 | P1 | Code Review 权限注解 |

---

## 三、修改范围（建议修改文件清单 ← 实际代码前先列出来让 AI 有边界感）

| 修改类型 | 目标文件路径 | 修改内容摘要 |
|---|---|---|
| 修改 | backend/crm/src/main/java/cn/cordys/crm/clue/service/ClueExportService.java | 重写 export 方法：分页查 + EasyExcel 流式写 |
| 修改 | backend/crm/src/main/java/cn/cordys/crm/clue/controller/ClueController.java | export 接口新增 options 参数，默认兼容旧版 |
| 新增 | （如需要） | |
| 配置 | （如需要） | |

---

## 四、必须遵守的技术约束（从知识库引用，不用重写细节）

| 约束项 | 知识入口 | 具体要求 |
|---|---|---|
| 功能权限 | knowledge/applications/backend/tech/security.md | Controller 写操作必须加 `@CsPermission("CLUE:EXPORT")` |
| 操作日志 | knowledge/applications/backend/tech/operation-log.md | 写操作必须加 `@OperationLog`，SpEL 绑定 resourceId/Name |
| 事务边界 | knowledge/applications/backend/tech/transaction.md | @Transactional 默认回滚 RuntimeException；手动抛出的异常用 GenericException |
| 导出（若有） | knowledge/applications/backend/tech/excel-io.md | 大导出必须异步 + 导出任务中心 + 分页/流式 |
| 数据权限 | knowledge/main/business-rules.md#1-3 | 查询必须经过 DataScopeService，避免越权 |
| （前端）API 调用 | knowledge/applications/lib-shared/INDEX.md | 用 @lib/shared/api/modules/clue.ts，不要 new axios |
| （前端）权限按钮 | knowledge/applications/frontend-web/INDEX.md#四 | 用 `v-permission` 或 `usePermission().hasPermission()` |

---

## 五、关键设计决策（方案选型，明确为什么这样做）

| 决策 | 选中方案 | 放弃方案 | 放弃原因 |
|---|---|---|---|
| 大导出实现方式 | 分页查询 + FastExcel 流式写（SXSSF） | 一次性全量查 + 普通写 | 10W+ 行 OOM 风险不可接受 |
| 异步导出任务中心 | 复用现有 system/export 体系 | 新建独立任务表 | 避免重复造轮子，已有任务通知/重试/下载链路 |

---

## 六、测试要点清单（提测前自测覆盖）

### 6.1 功能测试

- [ ] 同步导出（<1W）是否与历史行为一致
- [ ] 异步导出（>阈值）：是否正确写入导出任务中心，状态流转（进行中→成功/失败）
- [ ] 权限拦截：无 CLUE:EXPORT 的用户点击后 403

### 6.2 性能测试

- [ ] 10W 行导出：应用内存 <512MB，耗时 <30s
- [ ] 并发 3 人导出：互不影响，全部成功

### 6.3 兼容测试

- [ ] 旧前端（不带 options 参数）调用新接口：行为完全同旧版
- [ ] 旧导出模板列顺序与新版完全一致

### 6.4 异常测试

- [ ] DB 查询中途失败：正确抛异常，导出任务标记失败，不残留临时文件
- [ ] 用户容量不足（如有）：友好提示不返回 500

---

## 七、回滚方案（万一上线有严重问题时怎么做）

1. **回滚步骤**：
   - 后端：发布上一版本镜像（保留旧前端兼容）
   - 前端：同上（后端兼容旧前端，因此可单独回滚）
2. **数据补偿**：本需求无新增 DB 结构，回滚不需要做数据补偿
3. **已产生的导出任务记录**：保留历史即可，不影响业务

---

## 八、签字确认（进入编码前）

- [ ] 需求点清单完整可验证
- [ ] 修改范围明确，没有明显超出范围的文件
- [ ] 引用的知识库约束已读
- [ ] 测试要点覆盖关键场景
- [ ] 回滚方案可行
- [ ] 开发确认开始编码：@{user} / AI
