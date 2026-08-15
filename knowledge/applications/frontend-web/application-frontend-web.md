---
type: application
status: official
evidence: frontend/packages/web 源码
owner: cordys-crm
verify-required: true
verify-note: 实际组件/hooks/页面结构需回到代码核对
updated: 2026-08-11
---

# PC 端前端（frontend-web）应用总览

> 代码路径：`frontend/packages/web/`（@cordys/web）
> 技术栈：Vue 3.5 + TypeScript 5.9 + Vite + Naive UI + Pinia + Tailwind CSS + Less

---

## 一、应用职责

**负责什么**：
- CRM PC 端全部页面渲染与交互（浏览器访问 `/web` → index.html）
- 完整业务能力：线索/客户/商机/合同/订单/产品/仪表盘/审批/自定义表单/系统管理/智能体/标讯/工作台
- 国际化（中/英）、主题适配、权限指令、AI 聊天组件集成
- 导出（Excel/PDF）、打印、文件上传预览

**不负责什么**：
- 不负责移动端页面（frontend-mobile 负责）
- 不直接写 API 封装（统一从 @lib/shared/api 读取）
- 不直接定义枚举/TS 类型（统一从 @lib/shared/enums 和 @lib/shared/models 读取）

---

## 二、目录结构速查

```
frontend/packages/web/src/
├── main.ts                  # 应用入口 setupApp()
├── App.vue                  # 根组件：NConfigProvider + OAuth + SSE + 主题
├── router/                  # Vue Router（hash 模式）+ 路由守卫
│   ├── routes/modules/      # 按业务模块拆分路由（agent/clue/order/system...）
│   └── guard/               # permission + userLoginInfo 守卫
├── store/                   # Pinia stores
│   ├── modules/app/         # 应用态（index/types/visit）
│   ├── modules/user/        # 用户态（登录信息、权限、组织）
│   ├── modules/overview.ts  # 首页概览态
│   └── modules/view.ts      # 视图/列表持久态
├── config/                  # 11 个业务配置文件
│   ├── business.ts / clue.ts / contract.ts / follow.ts
│   ├── globalTask.ts / opportunity.ts / process.ts
│   ├── system.ts / workbench.ts / pathMap.ts
├── hooks/                   # 30+ 组合式函数（最常用：usePermission / useFormCreate*）
├── layout/                  # default-layout / full-page-layout / page-content
├── views/                   # 页面（按业务模块分目录）
│   ├── agent/               # 智能体管理
│   ├── base/login/          # 登录页
│   ├── clueManagement/      # 线索管理 + 线索池
│   ├── contract/            # 合同 + locale
│   ├── customForm/          # 自定义表单
│   ├── customer/            # 客户 + 联系人 + 公海 + locale
│   ├── dashboard/           # 仪表盘 + fullPage + link + module
│   ├── opportunity/         # 商机
│   ├── order/order/         # 订单
│   ├── product/             # 产品 + 价格
│   ├── system/              # 系统管理（business/log/message/module/org/role）
│   ├── tender/              # 标讯
│   └── workbench/           # 工作台 + smart(AI 聊天)
├── components/
│   ├── business/            # 业务组件（ai-chat / crm-approval / crm-flow X6
│   │                       #  crm-form-create / crm-follow / crm-stage-board
│   │                       #  crm-comment / crm-task / crm-city-select ...）
│   └── pure/                # 通用纯组件（crm-table / crm-chart*5
│                           #  crm-advance-filter / crm-tree / crm-upload
│                           #  crm-pagination / crm-drawer / crm-modal / crm-card）
├── locale/                  # 语言包（zh-CN / en-US，含 common + sys + settings）
└── utils/                   # export / permission / theme / themeOverrides
```

---

## 三、启动流程（main.ts）

```
setupApp()
  │
  ├─ createApp(App)
  ├─ 1. setupPinia stores
  ├─ 2. setupI18n(APP)  ← 阻塞加载语言包（从 localStorage 读 CRM-locale）
  ├─ 3. setupCrmIcon  ← 注册 crm-* SVG 图标
  ├─ 4. setupDayjsLocale  ← dayjs 语言同步
  ├─ 5. setupRouter + 路由守卫
  ├─ 6. setupLocalforage  ← 本地离线存储
  ├─ 7. setupDirective  ← v-permission + 指令
  └─ mount('#app')
```

---

## 四、业务模块 → 页面 / 组件 映射

| 业务模块 | views/ 目录 | 关键业务组件 components/business/ |
|---|---|---|
| 线索 | clueManagement/ | crm-follow / crm-form-create / crm-city-select |
| 客户+联系人+公海 | customer/ | crm-follow / crm-form-create / crm-tree |
| 商机 | opportunity/ | crm-stage-board / crm-formula + editor / crm-follow |
| 合同 | contract/ | crm-approval / crm-contract-timeline |
| 订单 | order/order/ | crm-stage-board / crm-form-create |
| 产品+价格 | product/ | crm-form-create / crm-table |
| 仪表盘 | dashboard/ | crm-chart*5 / crm-advance-filter / crm-card |
| 工作台+AI聊天 | workbench/smart/ | ai-chat（lib-shared 复用） |
| 审批流+审批画布 | system/ | crm-flow（AntV X6） / crm-approval |
| 自定义表单 | customForm/ | crm-form-create / crm-form-design |
| 智能体管理 | agent/ | components/business/tree.vue |
| 系统管理 | system/*/ | crm-table / crm-tree / crm-upload / crm-modal |

---

## 五、常用 Hooks 速查

| Hook 名 | 用途 | 高频场景 |
|---|---|---|
| `usePermission()` | 权限判断（hasPermission + 数据范围） | 所有页面渲染按钮/列时 |
| `useUser()` | 当前用户信息、组织切换 | 登录后信息读取 |
| `useFormCreateApi()` | 自定义表单引擎 API（读字段/校验/渲染） | 所有业务详情/编辑页 |
| `useFormCreateTable()` | 自定义表单 + 表格混排 | 列表页列配置 |
| `useFormReviewAction()` | 审批操作（通过/驳回/加签/转办） | 审批页 |
| `useApprovalOperation.ts` | 审批流启动、查看流程进度 | 单据审批详情页 |
| `useOpenDetailPage()` | 打开详情页（新标签/当前页/抽屉） | 列表行点击跳转 |
| `useOpenNewPage()` | 打开新标签页 | 打开外部或内部新标签 |
| `useModal()` | 通用对话框控制 | 所有二次确认/弹窗场景 |
| `useTableStore()` | 列表页状态持久化（分页/筛选/排序） | 所有列表页 |
| `usePathMap()` | path → 页面元数据映射 | 面包屑、标题渲染 |
| `useMenuTree()` | 菜单树生成 + 权限过滤 | 侧边栏渲染 |
| `useLocalForage()` | 本地离线存储读写 | 大数据本地缓存 |
| `useDiscreteApi()` | Naive UI discrete API（message/dialog/notification） | 所有页面消息提示 |
| `useLoading()` | 统一 loading 管理 | 异步操作加载态 |
| `useLeaveUnSaveTip()` | 离开页面未保存提示 | 编辑页防丢失 |
| `useFullScreen()` | 全屏切换 | 仪表盘全屏 |
| `useReasonConfig()` | 输单/驳回原因配置 | 输单驳回弹窗 |

---

## 六、AI 进入 PC 前端的推荐路径

```
第1步: 读 application-frontend-web.md（本文件）
第2步: 读 views/{module}/ 对应页面源码
       ├─ 先看 <script setup lang="ts"> 中的 hooks 引用
       ├─ 再看 <template> 中的组件使用
       └─ 参考同目录下的 locale/{zh-CN.ts, en-US.ts}（文案入口）
第3步: 查 components/business/ 下的业务组件源码
第4步: 查 hooks/useXxx.ts 复用逻辑
第5步: 查 config/{module}.ts 中的静态配置
第6步: 查 router/routes/modules/{module}.ts 路由定义
第7步: 写代码前：回到 @lib/shared 核对 API/枚举/TS 类型
```
