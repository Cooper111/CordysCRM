# Cordys CRM · 知识库读写规则

> **AI 必读**：本文件定义 AI 读写知识库时必须遵守的强制规则。违反这些规则可能导致业务错误或返工。

---

## 一、读取规则

### R1: 按需加载，禁止全量读取
- ❌ 禁止：一次性读取 knowledge/ 下所有文件
- ✅ 必须：通过 ROUTING 定位 → 进入 applications/{app}/INDEX → 按关键词定位到具体文件
- 原因：上下文窗口有限，无关知识会干扰判断，导致实现跑偏

### R2: 先路由，再深入
- 任何需求必须先读 `knowledge/ROUTING.md`
- 定位到具体应用后，先读对应 `applications/{app}/INDEX.md`
- 再根据 INDEX 导航，按需读取 domain 或 tech 下的具体文件

### R3: 事实等级判断（必须牢记）
| 等级 | 来源 | 含义 | 操作要求 |
|---|---|---|---|
| L1 事实 | 当前仓库的代码、配置、资源文件 | 最终事实来源 | 直接使用，无需额外核对 |
| L2 稳定知识 | knowledge/main/、knowledge/applications/ | 经过 review 的沉淀知识 | **接口签名/DTO/枚举/配置/字段必须回到代码核对** |
| L3 候选知识 | knowledge/candidate/ | 暂存待确认 | 仅作参考，引用前必须结合代码二次确认，且标注"来自候选知识，可信度待确认" |
| L4 个人经验 | knowledge/personal/ | 个人笔记 | 不作为团队事实引用，仅用于启发思路 |

### R4: 易变项强制核对
以下内容即使在正式知识库中有描述，**编码前也必须回到当前代码仓库核对**：
- Controller 接口签名（路径、方法、参数、返回值）
- DTO / Domain 类字段（名称、类型、注解）
- 状态枚举值（名称 + 编码）
- 配置项（commons.properties / cordys-crm.properties）
- 数据库表名和字段名（参考 Flyway 最新迁移脚本）
- 前端路由定义（router/routes）
- API URL 常量（lib-shared/api/requrls/）

---

## 二、写入规则

### W1: 新知识先写入 candidate/，禁止直接写入正式区
- 需求执行中沉淀的新经验：写入 `knowledge/candidate/`
- 必须附带：
  - **来源证据**：来自哪个需求、代码路径、聊天记录摘要
  - **可信度标记**：高 / 中 / 低
  - **待确认项**：哪些点还需要 owner review
- 只有经过 owner review 且被多次验证的内容，才可以移入 main/ 或 applications/

### W2: 严格使用 template/ 中的模板
- 写任何新知识前，先复制 template/ 下对应类型的模板
- 必须填写 YAML Front Matter 中的关键字段（type、status、evidence、owner、verify-required 等）
- 禁止自由发挥格式：避免今天写成散文、明天写成代码笔记

### W3: 知识粒度约束
- 一个知识文件只讲一个主题（一个流程 / 一个状态 / 一个规则 / 一个接口）
- 跨多个主题的内容拆分成多个文件，用链接互相关联
- 文件长度控制在 300 行以内（超过则拆分）

### W4: 易变项只写"定位入口"，不写细节
- 对于接口签名、枚举值、配置项等易变内容：
  - ✅ 写：对应的代码文件路径、类名、方法名
  - ✅ 写：如何在代码中搜索定位
  - ❌ 不写：具体参数列表（代码改了就过时，误导 AI）

---

## 三、知识引用规则

### C1: 引用必须标注来源
- 在 analysis.md、requirement.md、implementation-check.md 中引用知识库时，必须标注：
  ```
  参考知识：knowledge/main/core-process.md#L15-L30
  代码核对：backend/crm/src/main/java/cn/cordys/crm/clue/service/ClueService.java#L45
  ```

### C2: 冲突处理
- 如果知识库描述与当前代码不一致，**以当前代码为准**
- 同时在 candidate/ 中记录这个不一致，标记"知识库待更新"
- 不要在未确认的情况下修改正式知识库

---

## 四、RD 过程文件规则

### P1: 过程文件写入 rd/requirements/{requirementId}/
- 每个需求一个独立目录，目录名建议：`{日期}-{缩写}` 如 `20260811-clue-export-optimize`
- 过程文件包括：clarification.md / analysis.md / requirement.md / implementation-check.md / knowledge-backfill.md 等

### P2: 接续文件设计
- 每个 RD 目录必须包含 `continue-prompt.md`
- 作用：会话中断后，AI 读取此文件即可恢复上下文，不丢失之前的分析结论
- 内容：当前进度、已确认点、待办清单、下一步指令

---

## 五、违禁清单（绝对禁止）

| ❌ 禁止行为 | 原因 |
|---|---|
| 直接把 AI 生成的未经核对的内容写入 knowledge/main/ 或 applications/ | 错误知识比没有知识更危险 |
| 引用 candidate/ 或 personal/ 内容但不标注可信度 | 会把个人经验当成团队事实 |
| 基于知识库描述直接编码而不回到代码核对易变项 | 接口/枚举/配置变化快，知识库必然滞后 |
| 全量读取 knowledge/ 目录 | 上下文污染，导致实现跑偏 |
| 跳过 ROUTING 直接进入某个应用目录 | 容易漏掉跨应用的全局约束 |
