# Cordys CRM 知识库 · 总索引

> AI 进入知识库时**首先读取本文件**。通过本索引快速定位需要读取的知识文件，禁止全量扫描 knowledge/ 目录。

---

## 0. 强制读取顺序

进入知识库后，严格按照以下顺序加载（每一步判断后决定是否进入下一步）：

```
第1步: 读取 knowledge/ROUTING.md
       ↓ 根据需求关键词定位 → 业务域 / 应用 / 模块
第2步: 读取对应 applications/{app}/INDEX.md
       ↓ 根据应用内导航定位 → domain/product / domain/solution / domain/base / tech
第3步: 按需读取知识文件（只加载对当前问题有用的内容）
第4步: 回到代码仓库核对易变项（接口/枚举/配置/字段）
```

---

## 1. 第一优先级：路由与规则

| 文件 | 何时读取 | 内容摘要 |
|---|---|---|
| [ROUTING.md](file:///e:/工作/金数湾/CordysCRM/knowledge/ROUTING.md) | **每个需求必读取** | 关键词→业务域→应用→知识入口的映射路由表 |
| [KNOWLEDGE-RULES.md](file:///e:/工作/金数湾/CordysCRM/knowledge/KNOWLEDGE-RULES.md) | **每个需求必读取** | AI 读写知识库的强制规则，事实等级、核对要求 |

---

## 2. 第二优先级：通用业务知识（main/）

当需求涉及**跨模块业务概念**时读取：

| 文件 | 关键词触发 | 内容摘要 |
|---|---|---|
| [main/glossary.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/glossary.md) | 任何不确定术语含义时 | CRM 核心术语表（线索、客户、商机、合同、订单、公海、跟进等） |
| [main/core-process.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/core-process.md) | 涉及跨模块流程时 | 核心业务链路：线索→转客户→商机跟进→合同签订→订单执行→收款开票 |
| [main/state-defs.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/state-defs.md) | 涉及状态、状态流转时 | 各模块通用状态定义、流转条件、终态判断 |
| [main/business-rules.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/business-rules.md) | 涉及权限、数据范围、分配规则时 | 全局业务规则：权限模型、数据权限、容量、公海规则、审批触发 |
| [main/tech-constraints.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/tech-constraints.md) | 任何编码前必读 | 全局技术约束：Java版本、框架选型、字段加密、UID生成、前后端约定 |

---

## 3. 第三优先级：应用级知识（applications/）

当路由定位到具体应用后读取对应目录：

| 应用目录 | 路由关键词 | 核心职责 |
|---|---|---|
| [applications/backend/](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/INDEX.md) | 后端、Java、Spring、Service、Mapper、接口、数据库 | Java 21 + Spring Boot 后端，含 framework/crm/app 三模块 |
| [applications/frontend-web/](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/frontend-web/INDEX.md) | PC端、Web、Naive UI、页面、组件、前端 | Vue 3 + Naive UI PC 端，完整业务能力 |
| [applications/frontend-mobile/](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/frontend-mobile/INDEX.md) | 移动端、H5、Vant、手机端 | Vue 3 + Vant UI 移动端，精简业务能力 |
| [applications/lib-shared/](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/lib-shared/INDEX.md) | 共享包、API、枚举、模型、i18n、axios | 前端共享：@lib/shared，API封装/枚举/TS类型/i18n/工具方法 |
| [applications/installer/](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/installer/INDEX.md) | 部署、Docker、安装、配置、启动、运维 | 安装部署：Dockerfile、启动脚本、内置/外置服务切换 |

---

## 4. 候选知识与个人经验

| 目录 | 用途 | 读取条件 |
|---|---|---|
| [candidate/](file:///e:/工作/金数湾/CordysCRM/knowledge/candidate/INDEX.md) | 暂存待 review 的知识 | 当 main/applications 中找不到答案时，可检索 candidate 作为参考（但标注可信度待确认） |
| [personal/](file:///e:/工作/金数湾/CordysCRM/knowledge/personal/) | 个人经验与踩坑 | 仅供个人参考，不作为团队事实引用 |

---

## 5. 知识写作模板

| 模板文件 | 用途 |
|---|---|
| [template/application.md](file:///e:/工作/金数湾/CordysCRM/knowledge/template/application.md) | 新建应用总览文档 |
| [template/domain.md](file:///e:/工作/金数湾/CordysCRM/knowledge/template/domain.md) | 新建领域/产品能力文档 |
| [template/flow.md](file:///e:/工作/金数湾/CordysCRM/knowledge/template/flow.md) | 新建流程文档 |
| [template/state.md](file:///e:/工作/金数湾/CordysCRM/knowledge/template/state.md) | 新建状态定义文档 |
| [template/rule.md](file:///e:/工作/金数湾/CordysCRM/knowledge/template/rule.md) | 新建业务规则文档 |
| [template/tech.md](file:///e:/工作/金数湾/CordysCRM/knowledge/template/tech.md) | 新建技术规范文档 |
| [template/code-snippet.md](file:///e:/工作/金数湾/CordysCRM/knowledge/template/code-snippet.md) | 新建代码片段/示例文档 |

---

## 6. 配套 RD 流程

研发过程产物落在 `rd/` 目录下：

| 路径 | 用途 |
|---|---|
| [rd/](file:///e:/工作/金数湾/CordysCRM/rd/README.md) | RD 流程根目录，含流程说明与模板 |

---

## 7. 事实等级速查

| 来源 | 等级 | 代码核对要求 |
|---|---|---|
| 当前代码仓库 | ⭐⭐⭐ 事实来源 | **无需**核对（它本身就是事实） |
| main/ + applications/ 正式知识 | ⭐⭐ 稳定参考 | 接口/枚举/字段/配置**必须**回到代码核对 |
| candidate/ 候选知识 | ⭐ 候选参考 | **必须**结合代码二次确认，需谨慎引用 |
| personal/ 个人经验 | 仅供参考 | 不做事实引用，仅作启发 |
