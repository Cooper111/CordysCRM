# Cordys CRM · RD 研发流程

> RD（Research Development）是将研发过程落到文件里的人机协同工作流。
> 配套知识库：`knowledge/`（底座）
> 过程产物目录：`rd/requirements/{requirementId}/`（每个需求一个独立目录）

---

## 一、RD 解决的三个问题

传统研发中，需求分析、方案设计、代码实现的判断散落在：
- ❌ 即时通讯聊天记录 → **不可复用**（上次分析找不到了）
- ❌ 人的大脑记忆 → **不可 review**（决策过程无痕迹）
- ❌ 临时文档 / 会话上下文 → **不可接续**（换会话/换人，上下文断了）

RD 用 Markdown 文件把全过程写入 `rd/requirements/{id}/`，确保：**可复用、可review、可接续**。

---

## 二、完整 RD 流程与检查点

```
需求 / PRD / Bug / 变更
  │
  ├─▶ [L1 输入层] /rd:verify-prd ── 质量门禁1：PRD 合格性检查
  │                                  ▶ 不合格：返回补材料，不进入后续流程
  │
  ├─▶ /rd:work ── 路由命令（日常只用这一个就够，自动引导下一步）
  │
  ├─▶ /rd:clarify ── 需求澄清（结合知识库问出不确认点）
  │                    产物：clarification.md
  │
  ├─▶ [L2 分析层] /rd:analyze ── 需求分析
  │                    产物：analysis.md（背景/域/变更点/风险/依赖）
  │
  ├─▶ /rd:decompose ── 按应用拆解需求
  │                    产物：decomposition.yaml（多应用时每个应用自包含）
  │
  ├─▶ /rd:verify-requirement ── 质量门禁2：拆解确认（无遗漏+边界清）
  │
  ├─▶ 业务应用仓库实际开发（手动或 /rd:apply 自动）
  │
  ├─▶ [L3 实现层] /rd:validate ── 质量门禁3：需求-代码对比校验
  │                    产物：implementation-check.md
  │
  ├─▶ /rd:code-review ── 质量门禁4：AI 预审（过滤低级问题，人工 CR 聚焦架构）
  │
  ├─▶ /rd:release-plan ── 发布计划（顺序/灰度/回滚/监控）
  │
  ├─▶ 实际发布（CI/CD + 人工发布）
  │
  └─▶ 知识回补 ── 质量门禁5：经验沉淀（稳定知识→candidate/，个人经验→personal/）
       产物：knowledge-backfill.md
```

### 适用范围

| 需求类型 | 是否建议走完整 RD | 建议方式 |
|---|---|---|
| 跨多应用的复杂需求 | ✅ **强烈建议** | 完整跑所有检查点 |
| 单应用但业务状态/上下游协议复杂 | ✅ 建议 | 至少 clarify + analyze + validate |
| 中风险：历史兼容逻辑多/发布风险高 | ✅ 建议 | 至少 analyze + release-plan |
| 小修复：局部 Bug、纯样式调整、一次性脚本 | ❌ 不需要 | 直接改 + 人工 CR + 用 kb 命令简单入知识库 |
| 低风险工具需求 | ❌ 不需要 | 直接改 + review |

---

## 三、需求目录结构

每个需求对应：`rd/requirements/{requirementId}/`

**命名建议**（便于搜索和排序）：
```
{YYYYMMDD}-{模块缩写}-{3-5个词描述}
例：
  20260811-clue-export-optimize
  20260815-customer-recycle-bugfix
  20260901-approval-counter-sign
```

**目录内固定文件**：

```
rd/requirements/{requirementId}/
├── input/                        # 原始输入材料（只读归档）
│   ├── prd.md 或 prd.pdf         # 用户给的 PRD（如有）
│   ├── bug-report.md             # Bug 报告（如有）
│   ├── screenshot*.png           # 截图
│   └── conversation.md           # 关键聊天记录整理
├── materials/                    # 补充材料（分析中收集到的）
│   ├── related-code.md           # 关联代码路径清单
│   ├── related-knowledge.md      # 已读知识库文件清单
│   └── ...
├── clarification.md              # RD 产物1：需求澄清问答
├── analysis.md                   # RD 产物2：需求分析摘要
├── decomposition.yaml            # RD 产物3：按应用拆解
├── requirement.md                # RD 产物4：最终应用级需求文档（开发依据）
├── implementation-check.md       # RD 产物5：实现校验清单
├── continue-prompt.md            # RD 接续文件：换会话/换人时先读这个
└── knowledge-backfill.md         # RD 产物6：知识回补清单（→ candidate/ + personal/）
```

---

## 四、质量门禁三层

参考文章 "复杂业务团队的 AI Coding 交付实践" 5.2 节：

| 层级 | 检查点 | 目标 | 执行方式 |
|---|---|---|---|
| **L1 输入层** | PRD 验证（verify-prd） | 确保需求质量，带着模糊需求只会返工 | AI 自动检查清单 + 人工确认缺失项是否可接受 |
| **L2 分析层** | 澄清 → 分析 → 拆解确认 | 确保理解正确，不跑偏 | AI 生成 3 份 MD 文档 + 人工 review 结论合理性 |
| **L3 实现层** | 实现校验 + Code Review + 发布计划 | 确保实现质量 + 发布安全 | AI 自动对比需求点 vs 代码 + 人工把关关键判断 |

**人机分工原则**：
- 🤖 AI 负责：分析实现、生成代码、自动化检查、知识沉淀
- 👤 人聚焦：关键判断、架构决策、边界确认、异常处理

---

## 五、快速开始：一个最小 RD 示例

```bash
# 第1步：建目录
mkdir -p rd/requirements/20260811-clue-export-optimize/{input,materials}

# 第2步：写入 input/ 原始需求材料
# 第3步：AI 按模板生成 clarification.md + analysis.md
# 第4步：人工 review analysis.md（核心变更点和风险）
# 第5步：AI 生成 requirement.md（开发用）
# 第6步：实际编码（AI 或人）
# 第7步：AI 生成 implementation-check.md 校验
# 第8步：人工 CR + AI code-review 建议
# 第9步：生成 knowledge-backfill.md → 沉淀到 candidate/ + personal/
```

---

## 六、模板目录

`rd/template/` 下存放 RD 各阶段的 Markdown 模板，新建需求时复制对应模板改名。

| 模板文件 | 复制目标 | 说明 |
|---|---|---|
| [template/clarification.md](file:///e:/工作/金数湾/CordysCRM/rd/template/clarification.md) | requirements/{id}/clarification.md | 需求澄清问答模板 |
| [template/analysis.md](file:///e:/工作/金数湾/CordysCRM/rd/template/analysis.md) | requirements/{id}/analysis.md | 需求分析模板 |
| [template/decomposition.yaml](file:///e:/工作/金数湾/CordysCRM/rd/template/decomposition.yaml) | requirements/{id}/decomposition.yaml | 应用拆解模板 |
| [template/requirement.md](file:///e:/工作/金数湾/CordysCRM/rd/template/requirement.md) | requirements/{id}/requirement.md | 应用级需求模板（开发依据） |
| [template/implementation-check.md](file:///e:/工作/金数湾/CordysCRM/rd/template/implementation-check.md) | requirements/{id}/implementation-check.md | 实现校验模板 |
| [template/continue-prompt.md](file:///e:/工作/金数湾/CordysCRM/rd/template/continue-prompt.md) | requirements/{id}/continue-prompt.md | **必须有**：换会话恢复上下文 |
| [template/knowledge-backfill.md](file:///e:/工作/金数湾/CordysCRM/rd/template/knowledge-backfill.md) | requirements/{id}/knowledge-backfill.md | 知识回补清单 |

---

## 七、与知识库（knowledge/）的协作

- **RD 读取知识库**：澄清/分析阶段通过 ROUTING 定位知识，加载正确上下文（避免瞎猜）
- **RD 回写知识库**：需求完成后，把稳定经验 → `knowledge/candidate/`，个人经验 → `knowledge/personal/`
- **禁止**：RD 过程中 AI 直接写正式知识到 knowledge/main/ 或 applications/（必须走 candidate → review 流程）

---

## 八、为什么用 Markdown 文件而不是聊天记录？

| 维度 | 聊天记录 | RD Markdown 文件 |
|---|---|---|
| 可接续 | ❌ 新会话历史压缩，结论漂移 | ✅ `continue-prompt.md` 精准恢复 |
| 可复用 | ❌ 散落在多群/多人，检索困难 | ✅ 目录结构化，全仓库可检索 |
| 可 Review | ❌ 聊天里的判断没法 CR | ✅ MD 可 git diff，评审记录可追溯 |
| 可交接 | ❌ 换人要翻聊天 | ✅ 给 ID 读目录就知道全部上下文 |
| 可回归 | ❌ 下次类似需求从头分析 | ✅ 旧 analysis.md 直接参考类比 |
