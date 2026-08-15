---
type: application
status: official
evidence: frontend/packages/mobile 源码
owner: cordys-crm
verify-required: true
updated: 2026-08-11
---

# 移动端前端（frontend-mobile）应用总览

> 代码路径：`frontend/packages/mobile/`（@cordys/mobile）
> 技术栈：Vue 3.5 + TypeScript + Vite + **Vant UI** + pnpm + postcss-pxtorem（rem 适配）
> 访问路径：浏览器访问 `/mobile` → mobile/index.html

---

## 一、应用职责与功能差异

移动端是 PC 端的**精简子集**，聚焦销售在外移动办公场景：

| 功能 | PC Web | Mobile | 备注 |
|---|---|---|---|
| 线索 / 线索池 | ✅ | ✅ | 完整 |
| 客户 / 联系人 / 公海 | ✅ | ✅ | 完整 |
| 商机 | ✅ | ✅（无报价） | 报价功能暂未开放 |
| 合同 / 发票 / 收款 | ✅ | ❌ | |
| 订单 | ✅ | ❌ | |
| 产品 / 价格 | ✅ | ❌ | |
| 仪表盘 | ✅ | ❌ | |
| 自定义表单 | ✅ | ❌ | |
| 智能体独立页 | ✅ | ❌ | 含在工作台 AI 聊天 |
| 标讯 | ✅ | ❌ | |
| 系统管理（全部） | ✅ | ❌ | |
| 工作台（含AI聊天/审批/查重） | ✅ | ✅ | 完整 |
| 我的（消息/个人中心） | 含在个人中心 | ✅独立页 | Mobile 特色 |

---

## 二、与 Web 包的关键差异

| 维度 | Web（Naive UI） | Mobile（Vant UI） |
|---|---|---|
| UI 库 | Naive UI（桌面体验） | Vant 4（移动端体验） |
| 样式适配 | px（桌面分辨率） | **postcss-pxtorem** → rem（多屏幕适配） |
| Router base | `/`（根路径） | **`/mobile/`**（部署时加前缀） |
| VantResolver | ❌ | ✅（自动导入 Vant 组件） |
| 登录守卫 | 标准登录页 | 额外 AUTH_DISABLED_ROUTE / AUTH_LOADING_ROUTE（三方兼容） |
| 本地存储 | Pinia + localforage | Pinia（无 localforage） |
| 调试工具 | 普通 DevTools | 我的页连点 10 次用户名 → 唤起 Eruda |

**调试技巧**：
1. 先登录 PC Web，复制 localStorage 中 `sessionId` + `csrfToken`
2. 粘贴到 Mobile 页面对应 localStorage 项
3. 刷新即可直接进入登录态

---

## 三、目录结构

```
frontend/packages/mobile/src/
├── main.ts / App.vue          # 入口 +  Suspense 包裹（先 changeLocale）
├── config/
│   ├── follow.ts              # 跟进配置（复用 lib-shared 枚举）
│   └── mine.ts                # 我的页配置
├── router/
│   ├── routes/modules/        # clue / customer / mine / opportunity / workbench
│   └── guard/permission.ts    # 登录守卫
├── store/                     # app + setting + user（与 web 同结构）
├── hooks/                     # useFormCreateApi / useHiddenTab / useLogin(mobile专用) / useUser
├── views/
│   ├── base/login / no-resource
│   ├── clue/ (clue + pool)
│   ├── customer/ (detail + index + relation + transfer)
│   ├── mine/ (detail + index + message + locale)
│   ├── opportunity/
│   └── workbench/ (含 AI聊天 + 审批 + 查重 + 跟进 + 任务)
└── locale/ (zh-CN / en-US)
```

---

## 四、AI 进入移动端的推荐路径

```
1. 读本文件 application-frontend-mobile.md
2. 确认功能是否在移动端支持（上表），不支持则不要硬实现
3. 对应 views/{module}/ 读源码
4. 查 Vant 官方文档 + VantResolver 自动导入规则
5. 共享内容：API/枚举/类型/i18n → @lib/shared（与 web 共用）
```
