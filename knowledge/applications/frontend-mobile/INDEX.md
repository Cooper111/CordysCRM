# 移动端前端 · 索引（INDEX）

> 进入 @cordys/mobile 后首先读取。

---

## 一、views/ 页面导航

| 目录 | 功能 |
|---|---|
| [views/base/login/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/mobile/src/views/base/login/) | 登录页（账密+OAuth） |
| [views/clue/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/mobile/src/views/clue/) | 线索(clue) + 线索池(pool) |
| [views/customer/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/mobile/src/views/customer/) | 客户列表/详情/联系人/公海/移交 |
| [views/opportunity/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/mobile/src/views/opportunity/) | 商机（无报价） |
| [views/mine/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/mobile/src/views/mine/) | 我的：消息/详情/设置 + locale |
| [views/workbench/](file:///e:/工作/金数湾/CordysCRM/frontend/packages/mobile/src/views/workbench/) | 工作台：AI 聊天/审批/查重/跟进/任务 |

---

## 二、Hooks 说明

| Hook | Mobile 特有 | 说明 |
|---|---|---|
| `useLogin` | ✅ 是 | 移动端专用登录流程（含三方兼容） |
| `useHiddenTab` | ✅ 是 | 页面底部 tabbar 显示/隐藏控制 |
| `useFormCreateApi` | ❌ 否 | 与 web 同，复用自 lib-shared 封装 |
| `useUser` | ❌ 否 | 与 web 同结构 |

---

## 三、与 Web 共享清单

✅ **全部共享**（来自 @lib/shared）：
- API 请求封装（api/http + api/modules）
- 枚举定义（clueEnum/customerEnum/opportunityEnum/...）
- TS 类型模型（models/*）
- i18n 核心（locale/setupI18n）
- ai-chat 组件
- 工具方法（auth/format/dom/validate...）
