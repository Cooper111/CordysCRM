# Cordys CRM 知识库

> 本知识库为 Cordys CRM 项目的 AI Coding 交付底座，沉淀业务上下文、研发规范和历史经验，确保 AI 在正确上下文中工作，减少返工，提升交付质量。
>
> - 仓库根目录: `e:\工作\金数湾\CordysCRM`
> - 知识库路径: `knowledge/`
> - 配套 RD 流程: `rd/`
> - 上游开源: https://github.com/1Panel-dev/CordysCRM

---

## 核心原则

1. **知识分层**: 区分全局知识(main)、应用知识(applications)、候选知识(candidate)、个人经验(personal)，建立清晰流转机制
2. **路由优先**: 不做全量读取，通过 ROUTING 按需定位、渐进加载上下文
3. **过程落盘**: 研发过程通过 Markdown 文件落到 rd/ 目录，确保可复用、可 review、可接续
4. **质量前置**: 关键节点设置检查点，避免后期高成本返工
5. **人机协同**: AI 负责分析实现，人聚焦关键判断

---

## 目录结构

```
knowledge/
├── main/                 # 跨应用通用业务知识
│   ├── glossary.md       # 核心术语表
│   ├── core-process.md   # 核心业务流程（线索→客户→商机→合同→订单→收款）
│   ├── state-defs.md     # 通用状态定义与流转
│   ├── business-rules.md # 全局业务规则
│   └── tech-constraints.md # 全局技术约束
├── applications/         # 应用级知识（按应用分目录）
│   ├── backend/          # 后端应用总览
│   ├── frontend-web/     # PC 端前端
│   ├── frontend-mobile/  # 移动端前端
│   ├── lib-shared/       # 前端共享包
│   └── installer/        # 部署/安装器
├── candidate/            # 候选知识暂存区（待 review 的知识）
│   └── INDEX.md
├── personal/             # 个人经验与踩坑记录
│   └── .gitkeep
├── template/             # 强约束的知识写作模板
│   ├── application.md
│   ├── domain.md
│   ├── flow.md
│   ├── state.md
│   ├── rule.md
│   ├── tech.md
│   └── code-snippet.md
├── INDEX.md              # 知识库总索引（AI 进入时首先读取）
├── README.md             # 本文件，知识库说明
├── KNOWLEDGE-RULES.md    # 知识库读写规则（AI 必须遵守）
└── ROUTING.md            # 知识路由规则（需求→知识入口的映射）
```

---

## 知识流转机制

```
personal 个人经验
    ↓ (经验整理 + 证据补充)
candidate 候选知识
    ↓ (owner review + 代码核对)
main/ 或 applications/ 正式知识
    ↓ (需求执行中被引用)
    ↓ (代码或业务变化时)
更新 / deprecated 标记
```

**重要**: 接口签名、DTO 字段、状态枚举、开关配置等易变内容，即使在知识库中存在，也仅作为**定位入口**。真正改代码前必须回到当前代码仓库核对。

---

## 快速开始（AI 使用指南）

AI 进入本仓库后，应按以下顺序操作：

1. **读 ROUTING.md** → 根据需求关键词定位业务域和应用
2. **读 applications/{app}/INDEX.md** → 进入应用级导航
3. **按需加载** → application总览 → product主干 → solution差异 → base索引 → tech规范
4. **回到代码核对** → 接口/枚举/配置等易变项必须核对当前代码
5. **知识回补** → 需求完成后，将稳定经验写入 candidate/ 待 review
