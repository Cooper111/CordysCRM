# PC 端前端（frontend-web）知识 · 索引（INDEX）

> AI 进入前端 web 包后首先读取本文件。

---

## 一、读取顺序

```
1. 读 application-frontend-web.md（应用总览，已读）
2. 定位业务模块 → views/{module}/ 源码 + 就近 locale
3. 查复用逻辑 → hooks/useXxx.ts + components/business/
4. 查配置与路由 → config/{module}.ts + router/routes/modules/{module}.ts
5. 查 API/枚举/类型 → @lib/shared（applications/lib-shared/INDEX.md）
```

---

## 二、views/ 页面目录导航

| 页面目录 | 功能说明 | 关键子文件 |
|---|---|---|
| [views/agent/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/agent/) | 智能体管理页 | index.vue + components/tree.vue + locale/* |
| [views/base/login/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/base/login/) | 登录页 | index.vue（支持账密 + OAuth 登录） |
| [views/clueManagement/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/clueManagement/) | 线索管理 + 线索池 | clue + pool 各自 index/detail |
| [views/contract/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/contract/) | 合同管理 | locale/*（中/英文案） |
| [views/customForm/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/customForm/) | 自定义表单设计 + 数据 | index.vue |
| [views/customer/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/customer/) | 客户 + 联系人 + 公海 | customer.vue + contact.vue + openSea.vue + locale/* |
| [views/dashboard/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/dashboard/) | 仪表盘 | index.vue + fullPage.vue + link.vue + module.vue + locale/* |
| [views/opportunity/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/opportunity/) | 商机管理 | index.vue（阶段看板/列表切换） |
| [views/order/order/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/order/order/) | 订单管理 | index.vue + locale/* |
| [views/product/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/product/) | 产品 + 价格 | index.vue + price.vue + locale/* |
| [views/system/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/system/) | 系统管理（6 子目录） | business/log/message/module/org/role 各 index.vue |
| [views/tender/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/tender/) | 标讯管理 | index.vue |
| [views/workbench/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/web/src/views/workbench/) | 工作台 + AI 聊天 | index.vue + smart/index.vue + locale/* |

---

## 三、components/business/ 关键业务组件

| 组件目录/名 | 功能说明 | 复用页面 |
|---|---|---|
| `crm-approval/` | 审批通用组件（审批页、审批详情、审批操作） | 合同/订单/工商抬头/赢单/输单审批 |
| `crm-flow/` | 审批流画布（AntV X6） | 审批流设计页（system） |
| `crm-form-create/` | 自定义表单引擎（渲染+编辑） | 所有业务详情/编辑页 |
| `crm-formula + editor/` | 公式编辑器 | 商机阶段规则、字段公式配置 |
| `crm-follow/` | 跟进计划/记录组件 | 线索/客户/商机/合同详情页 |
| `crm-stage-board/` | 阶段看板（商机/订单） | 商机列表页看板视图、订单列表页 |
| `crm-status-config/` | 状态/阶段配置组件 | 商机阶段、订单阶段配置 |
| `crm-comment/` | 评论组件 | 跟进记录评论、单据评论 |
| `crm-task/` | 任务组件（跟进/导出任务） | 个人中心、工作台 |
| `crm-city-select/` | 省市区三级联动（region.json） | 地址字段 |
| `ai-chat/` | AI 聊天组件（复用自 lib-shared） | 工作台/smart、智能体页面 |

---

## 四、components/pure/ 通用纯组件

| 组件名 | 说明 |
|---|---|
| `crm-table` | 列表表格（Naive UI 封装 + 列配置 + 筛选 + 分页） |
| `crm-chart`(5 种) | 折线/柱状/饼/雷达/漏斗图（echarts 封装） |
| `crm-advance-filter` | 高级筛选器（与 crm-table 配合） |
| `crm-tree` | 组织/部门/分类树（Naive Tree 封装） |
| `crm-upload` | 文件上传（FileCenter 协议） |
| `crm-pagination` | 分页器（与 crm-table 配合） |
| `crm-drawer` | 抽屉（详情页常见） |
| `crm-modal` | 对话框（统一封装 + 拖拽） |
| `crm-card` | 卡片容器（仪表盘、工作台用） |
| `crm-tag` | 状态标签（按配置色渲染） |

---

## 五、hooks/ 快速定位

知道 Hook 名时直接去：`frontend/packages/web/src/hooks/useXxx.ts`

| 类别 | 代表 Hook |
|---|---|
| 权限 | usePermission |
| 表单引擎 | useFormCreateApi / useFormCreateTable / useFormReviewAction |
| 审批 | useApprovalOperation |
| 页面跳转 | useOpenDetailPage / useOpenNewPage / usePathMap |
| 状态/存储 | useTableStore / useLocalForage / useMenuTree |
| UI 辅助 | useModal / useDiscreteApi / useLoading / useFullScreen |
| 其他 | useLeaveUnSaveTip / useReasonConfig / useUser / useVisit |

---

## 六、config/ 业务配置入口

| 配置文件 | 内容 |
|---|---|
| `config/business.ts` | 全局业务配置（业务开关/默认值） |
| `config/clue.ts` | 线索配置（默认列/状态/查询条件） |
| `config/contract.ts` | 合同配置 |
| `config/follow.ts` | 跟进配置 |
| `config/globalTask.ts` | 全局任务配置 |
| `config/opportunity.ts` | 商机配置（阶段/阶段推进必填） |
| `config/process.ts` | 审批流默认配置 |
| `config/system.ts` | 系统管理配置 |
| `config/workbench.ts` | 工作台配置 |
| `config/pathMap.ts` | 路由路径 → 面包屑/标题映射 |

---

## 七、与共享包的协作边界

| 职责 | 归属包 | 路径 |
|---|---|---|
| API 请求封装 | @lib/shared | `lib-shared/api/modules/*` + `lib-shared/api/requrls/*` |
| 枚举定义 | @lib/shared | `lib-shared/enums/*` |
| TS 类型/模型 | @lib/shared | `lib-shared/models/*` |
| 工具方法（加解密/格式化） | @lib/shared | `lib-shared/method/*` |
| i18n 核心 setupI18n | @lib/shared | `lib-shared/locale/*` |
| ai-chat 组件 | @lib/shared | `lib-shared/ai-chat/*` |
| 页面/业务组件 | web（本包） | `src/views/` + `src/components/` |
| Hooks（页面级） | web（本包） | `src/hooks/` |
| 业务配置 | web（本包） | `src/config/` |
