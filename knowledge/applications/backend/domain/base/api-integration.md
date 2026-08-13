---
type: domain-base
sub-type: api-index
module: integration
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/integration/ 目录（或各模块下 controller 中 @RequestMapping 含 /maxkb /dataease /im /qcc /sso 等）
owner: cordys-crm
verify-required: true
verify-note: 第三方 API token、回调地址、字段映射 极易变，核对代码和配置中心
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/integration.md
---

# 三方集成 模块 API 索引（api-integration）

---

## 一、Controller 类定位（按集成点分）

### 1. MaxKB 知识库集成

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **MaxKbController**（或命名为 KbController） | MaxKB 文档同步：按模块（线索/客户）批量推送数据到 MaxKB、手工触发单条同步、查看同步状态、配置同步字段映射 | MaxKbIntegrationService + SyncTaskService | `INTEGRATION:MAXKB_MANAGE / SYNC` |

### 2. DataEase 可视化集成

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **DataEaseController** | DataEase 仪表板嵌入：按用户生成带签名的 embed URL、DataEase 数据集同步接口（CRM→DataEase push）、DataEase 资源清单查询 | DataEaseIntegrationService + DataEaseSignService | `INTEGRATION:DATAEASE_EMBED / DATAEASE_SYNC` |

### 3. IM 平台（钉钉/飞书/企微）集成

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **DingTalkController** | 钉钉：回调接收（事件订阅）、发送 IM 消息/卡片、钉钉用户同步到 CRM 本地用户、钉钉审批流回调 | DingTalkIntegrationService + DingTalkCallbackService | `INTEGRATION:DINGTALK_MANAGE / CALLBACK` |
| **LarkController**（或 FeishuController） | 飞书：同上（回调、IM 消息、审批、通讯录同步） | LarkIntegrationService + LarkCallbackService | `INTEGRATION:LARK_MANAGE / CALLBACK` |
| **WeComController**（WechatWork） | 企微：同上 | WeComIntegrationService + WeComCallbackService | `INTEGRATION:WECOM_MANAGE / CALLBACK` |

### 4. QCC（企查查）企业查询

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **QccController** | QCC 企业搜索（按名称/统一社会信用代码）、根据 QCC 企业 ID 抓取工商详情"一键创建客户"、QCC 调用配额统计 | QccIntegrationService + QccCustomerTransformService | `INTEGRATION:QCC_SEARCH / QCC_CREATE_CUSTOMER / QCC_QUOTA` |

### 5. SSO（统一身份认证）

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **SsoController** | SSO 登录入口（OAuth2/OIDC/CAS/SAML 具体哪种核对代码）、SSO 回调、本地 user 绑定 SSO ID、登出回调 | SsoIntegrationService（OAuth2AuthorizationCodeService 等） + 对应 SSO Provider | `INTEGRATION:SSO_MANAGE`（一般不需要权限，登录公开接口） |

### 6. 招标/投融资/数据服务 集成（如有）

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **TenderController**（如有） | 招标查询：按关键词/地区/行业抓取商机、一键转 CRM 线索/商机 | TenderIntegrationService + TenderTransformService | `INTEGRATION:TENDER_QUERY / TRANSFORM` |

---

## 二、定位技巧

| 关键词 | 找哪个集成点 Controller |
|---|---|
| 知识库 / AI 问答 / MaxKB / 同步文档 | MaxKbController |
| 图表嵌入 / DataEase / 数据集同步 | DataEaseController |
| 钉钉 / 飞书 / 企微 → 发消息、审批回调、通讯录同步 | DingTalkController / LarkController / WeComController |
| 企查查 / 企业查询 / 一键创建客户 / 工商信息 | QccController |
| SSO / 免登 / OAuth / 统一认证 / 登录跳转 | SsoController |
| 招标 / 采购公告 / 一键转商机 | TenderController（核对） |

路径前缀（核对 @RequestMapping）：
```
/api/integration/maxkb/*    → MaxKB
/api/integration/dataease/* → DataEase
/api/integration/dingtalk/* → 钉钉
/api/integration/lark/*     → 飞书
/api/integration/wecom/*    → 企微
/api/integration/qcc/*      → QCC
/api/sso/*                  → SSO（常放在根路径而非 integration 下）
/api/integration/tender/*   → 招标
```

## 三、配置（YML）定位

所有集成的 token / app_key / 回调地址在 `backend/crm/src/main/resources/application-*.yml` 的 `cordys.integration.*` 节点下（或对应配置中心）：
```yaml
cordys:
  integration:
    maxkb:
      base-url: ...
      api-key: ...
    dataease:
      base-url: ...
      sign-secret: ...
    dingtalk:
      app-key: ...
      app-secret: ...
      callback-token: ...
    lark: ...
    wecom: ...
    qcc:
      app-key: ...
      secret-key: ...
    sso:
      provider: oidc  # oauth2/cas/saml
      client-id: ...
```
**生产环境严禁把真实密钥写入知识库！**（安全准则 R2-S）
