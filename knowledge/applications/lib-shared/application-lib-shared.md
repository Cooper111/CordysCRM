---
type: application
status: official
evidence: frontend/packages/lib-shared 源码
owner: cordys-crm
verify-required: true
updated: 2026-08-11
---

# 前端共享包（lib-shared）应用总览

> 代码路径：`frontend/packages/lib-shared/`（包名 `@lib/shared`）
> 这是前端 monorepo 中**唯一被 web 和 mobile 共同依赖**的包，是前端底座的底座。

---

## 一、职责清单（必须死守边界）

所有"跨端共用"的内容必须放在这里，**禁止在 web/mobile 中重复定义**：

| 子目录 | 职责 | 典型内容 |
|---|---|---|
| `api/http/` | Axios 核心封装 | CordysAxios、AxiosCanceler、AxiosTransform、helper |
| `api/modules/` | 按业务模块的 API 函数 | clue / customer / opportunity / contract / order / product / follow / dashboard / home / customForm / sys / agent / ai 共 13+ 模块 |
| `api/requrls/` | URL 常量（与 modules 一一对应） | `{module}/index.ts` 导出具名 URL 常量 |
| `enums/` | 所有前端枚举 | clueEnum / customerEnum / opportunityEnum / contractEnum / formDesignEnum / httpEnum / moduleEnum / tableEnum / formula / process / systemEnum / uploadEnum / commonEnum |
| `models/` | TS 类型定义（业务模型） | clue / customer / system(子包 agentTask/aiModel/business/log/...) + 各模块单文件 |
| `method/` | 纯工具函数（无 Vue 依赖） | auth / comment / dom / equal / exportPdf / formCreate / index / is / local-storage / route-listener / scriptLoader / setupDrag / validate |
| `locale/` | i18n 核心 | setupI18n / useLocale / loadLocalePool（**不是语言包内容**，语言包内容就近在 web/mobile 各页定义） |
| `hooks/` | 跨端共用的 Hooks | `useI18n.ts` |
| `ai-chat/` | AI 聊天组件（@ai-sdk/vue） | AiChatProvider.vue + runtime/types + utils |
| `types/` | TS 全局类型声明 | axios.d.ts / global.d.ts |

---

## 二、API 封装协作模式（最重要）

```
api/requrls/clue/index.ts        ← URL 常量（纯字符串）
       │  export const GET_CLUE_LIST = '/api/clue/list'
       ▼
api/modules/clue.ts              ← 业务 API 函数（调用 CordysAxios）
       │  import { GET_CLUE_LIST } from '../requrls/clue'
       │  export const getClueList = (params) => CordysAxios.get(GET_CLUE_LIST, { params })
       ▼
web/src/views/... 或 mobile/src/views/...
       │  import { getClueList } from '@lib/shared/api/modules/clue'
       ▼
       直接调用
```

---

## 三、AI 修改本包的影响范围判断

| 修改的子目录 | 影响范围 | 回归测试建议 |
|---|---|---|
| `api/http/` | ⚠️ **全部前端请求** | web + mobile 全量冒烟 |
| `api/modules/*` + `api/requrls/*` | 对应业务模块 | 该模块 web + mobile 页面 |
| `enums/` | 对应模块状态/类型展示 | 所有引用该枚举的页面 |
| `models/` | 对应模块 TS 类型检查 | 编译即校验 |
| `method/auth` | 登录/加解密相关 | 登录流程 + 加密字段 |
| `ai-chat/` | AI 聊天组件 | 工作台 AI 聊天页 + 智能体页 |
| `locale/` | i18n 切换 | web + mobile 语言切换流程 |

---

## 四、快速定位

| 查找需求 | 目录入口 |
|---|---|
| 某个 API 的定义 | `api/modules/{module}.ts` + `api/requrls/{module}/index.ts` |
| 某个状态枚举的可能值 | `enums/{module}Enum.ts` |
| 某个 TS 类型的字段 | `models/{module}/` 或 `models/{module}.ts` |
| HTTP 拦截器/鉴权/取消请求 | `api/http/Axios.ts` + `axiosTransform.ts` + `axiosCancel.ts` |
| AI 聊天组件实现 | `ai-chat/AiChatProvider.vue` |
| RSA/AES 前端加密 | `method/auth.ts` |
| PDF 导出工具 | `method/exportPdf.ts` |
