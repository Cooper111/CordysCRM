---
type: domain-product
app: backend
module: integration
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/integration 下 10 个子包：agent(MaxKB) + dataease(DataEase) + dingtalk + lark(飞书) + wecom(企微) + qcc(企查查) + sso + tender(招投标) + sync(通用部门同步) + common（第三方通用配置/HTTP 客户端工具）
owner: cordys-crm
verify-required: true
verify-note: 第三方 API 路径、Token 刷新周期、接口返回结构、频率限制 极易变化，**必须回代码 + 对应官方文档最新核对**
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#集成/SSO/OAuth/智能体/标讯
  - knowledge/applications/backend/domain/base/api-integration.md
  - knowledge/applications/backend/application-backend.md 三、上下游（下游列表）
  - knowledge/applications/installer/application-installer.md（第三方配置变量）
---

# 第三方集成模块 · 产品能力

---

## 一、能力概述

把 CRM 与企业内外部系统打通，共 **10 个子域**，覆盖：

| 子域 | 目标系统 | 核心能力 |
|---|---|---|
| **agent** | MaxKB 智能体平台 | AI 智能体（Agent）+ 智能体模块（分类/收藏夹）；MCP/AI 对话调用 MaxKB 应用/工作流 |
| **dataease** | DataEase 开源 BI 平台 | 部门/角色/用户同步（Cordys→DataEase）、系统变量同步、报表嵌入 |
| **dingtalk** | 钉钉 | 部门同步、消息推送、扫码登录（配合 SSO）、通讯录获取 |
| **lark** | 飞书（Lark） | 部门同步、消息推送、审批回传、扫码登录（SSO） |
| **wecom** | 企业微信（WeCom） | 部门同步、消息推送、用户 ticket 获取（扫码 SSO） |
| **qcc** | 企查查 | 企业模糊查询、企业详情、招投标（标讯）查询 |
| **sso** | 通用 OAuth2 SSO | 第三方登录（钉钉/飞书/企微扫码 + 通用 OAuth）、Token 管理、OAuth 状态保护 |
| **tender** | 招投标/标讯聚合服务 | 标讯查询、标讯详情（后端封装 qcc.tender 或第三方） |
| **sync** | 通用部门/用户同步 | ThirdDepartmentService 抽象 + 各 IM 平台部门同步实现（管理后台统一触发） |
| **common** | 通用 | 所有第三方的 HTTP 客户端封装（HttpClientUtils/CSHttpClient）、通用配置 DTO（ThirdConfigBaseDTO）、OAuthUserService、二维码（QrCodeClient）、数据处理工具 |

设计模式：每个第三方 = 独立子包，结构统一：
`constants(ApiPaths) / dto(请求+响应) / response(统一响应包装) / service(业务封装) / client(HTTP 客户端) / controller(对外接口，部分有)`

---

## 二、核心流程（ASCII 全景图）

```
 ┌─────────────────────────────────────────────────────────────────────────┐
 │                          Cordys CRM 内部（调用方）                         │
 │  business modules / system.notice / management-center UI / Agent UI      │
 └────┬───────────────┬───────────────┬───────────────────┬───────────────┬──┘
      │ 通知/消息推送   │ 用户/部门同步   │ 企业/标讯查询     │ OAuth 登录      │ AI 智能体
      ▼               ▼               ▼                   ▼               ▼
 ┌────┴────┐    ┌──────┴─────┐   ┌─────┴─────┐     ┌──────┴──────┐   ┌────┴────┐
 │ DingTalk │    │ DataEase   │   │    QCC    │     │     SSO     │   │  Agent  │
 │  NoticeSender│ │  SyncService │ │  QccClient │     │ SSOService  │   │ MaxKB   │
 │ + DeptSync │   │  + Variables │ │ + Tender  │     │ + OAuthState│   │  Apps + │
 └────┬─────┘    └─────────────┘   └─────┬─────┘     └──────┬──────┘   │ Workflow│
      │ 部门同步                           │ 查企业          │ OAuth 回调   └────┬────┘
      ▼               ┌───────────────────┐                  │                 │
 ┌────┴────┐          │  ThirdDepartment  │     ┌────────────┴─────┐       调用
 │  Lark   │          │  Service (通用)   │     │ QrCodeClient     │   MaxKB API
 │ 飞书    │          │  (统一抽象)       │     │ 扫码登录二维码   │
 │  Notice │          └────────┬──────────┘     └──────────────────┘
 │ +DeptSync│                 │
 └────┬────┘           ┌──────┴────────┐
      │ 部门同步         │   common      │
 ┌────┴────┐           │ 通用工具：      │
 │ WeCom   │           │ HttpClientUtils │
 │ 企微    │           │ ThirdConfig*   │
 │ Notice │           │ OAuthUserService│
 │ +DeptSync│          │ DataHandleUtils│
 └─────────┘           └────────────────┘
```

### 子域详解（每个子域 1 段 + 代码入口）

| 子域 | 主要能力 | 核心类/代码入口（定位用，实现细节核对） | 关键配置 |
|---|---|---|---|
| **agent (MaxKB)** | 智能体 CRUD + 模块分类 + 收藏夹 + MaxKB 应用列表/工作流查询 + 脚本执行（ScriptRequest→ScriptResponse，调用 MaxKB 的应用/工作流 API） | AgentController + AgentModuleController；AgentBaseService、AgentModuleService；MaxKBConfigDetailDTO + constant/MaxKBApiPaths.java + response/*（MaxKB API 返回结构） | MaxKBThirdConfigRequest：baseUrl + apiKey + appId 等；integration 配置页维护 |
| **dataease** | DataEase 用户/角色/部门/系统变量同步（Cordys→DataEase），用户/角色 CRUD、系统变量/变量值 CRUD、资源挂载（DeTempResourceDTO：资源临时授权） | DataEaseService（CRUD）+ DataEaseSyncService（同步全量/增量）+ DataEaseClient（HTTP 客户端封装）；constant/DataScopeVariable + DataScopeDeptVariable；DTO request / response 全套 | DeThirdConfigRequest：DataEase baseUrl + ak/sk + 同步配置 |
| **dingtalk** | 钉钉消息推送（DingTalkNoticeSender）+ 部门同步（DingTalkDepartmentService，调用钉钉通讯录接口同步部门）+ 扫码登录用户查询（DingTalkUser） | DingTalkDepartmentService + DingTalkNoticeSender；constant/DingTalkApiPaths.java；dto/ DingTalk*；response/ DingTalk*Response 封装钉钉返回体 | DingTalkThirdConfigRequest：corpId + appKey + appSecret + 消息模板 agentId |
| **lark (飞书)** | 飞书消息推送 LarkNoticeSender + 部门同步 LarkDepartmentService + 审批回传 + 扫码登录用户查询（LarkUserResponse） | constant/LarkApiPaths.java；LarkDepartmentService / LarkNoticeSender；dto/Lark*（部门/消息/Token/租户/用户）+ response/Lark*Response | LarkThirdConfigRequest：appId + appSecret + tenantAccessToken 缓存 |
| **wecom (企微)** | 企微消息推送 WeComNoticeSender + 部门同步 WeComDepartmentService + 扫码登录 WeComUserTicketDTO（用 ticket 换 user 信息） | constant/WeComApiPaths.java；WeComDepartmentService / WeComNoticeSender； dto/WeCom*（部门/消息/Token/用户）+ response WeCom*Response | WecomThirdConfigRequest：corpId + agentId + secret + 消息 token + encodingAesKey（回调如有） |
| **qcc (企查查)** | 企业模糊搜索、企业详情（工商信息）、招投标（标讯）查询；封装 QccDetailDTO/QccEnterpriseInfo 等；调用企查查 API | constant/QccApiPaths.java；QccClient（HTTP 客户端，统一签名）；dto/qcc 下全套（Area/BankInfo/ContactInfo/EnterpriseInfo/Industry/InfoData/QccDetailDTO...）；response QccBaseResponse | QccThirdConfigRequest：key + secret（或 token，按实际对接方式核对） |
| **sso (SSO)** | SSO 通用登录：/login/oauth2/code/* 回调（SSOController） + OAuthStateService（state 校验，防 CSRF） + TokenService（第三方 Token 换成本地 Session）+ SSOService（综合封装） | SSOController；SSOService；OAuthStateService；TokenService；constant/OAuthStateFlow；integration.common.OAuthUserService 合并本地用户 | OAuth 配置项：各平台 clientId + clientSecret + redirectUri（在 installer cordys-crm.properties 或 第三方配置页维护） |
| **tender (招投标)** | 标讯查询 + 详情；实际调用路径 = 后端封装 qcc.tender（见 qcc.constant.TenderApiPaths）或独立标讯源 | TenderController（对前端的标讯查询）；dto/TenderDetailDTO + qcc.constant.TenderApiPaths | 同 QCC 配置或独立 TenderThirdConfigRequest |
| **sync (通用部门同步)** | ThirdDepartmentService 抽象 + ThirdDepartment + ThirdUser + ThirdOrgDataDTO（统一结构），把三端（钉钉/飞书/企微）的部门同步统一结构；ThirdSwitchLogDTO（同步开关日志） | sync.service.ThirdDepartmentService + sync.dto.*（ThirdDepartment / ThirdUser / ThirdOrgDataDTO / ThirdSwitchLogDTO） | 各平台"部门同步开关"在组织配置中维护 |
| **common (通用)** | ① HTTP 客户端封装（HttpClientUtils + CSHttpClient）；② 通用配置基类（ThirdConfigBaseDTO）+ 8 种 XXXThirdConfigRequest；③ OAuthUserService（OAuth 后合并本地用户）；④ QrCodeClient（扫码登录生成二维码）；⑤ DataHandleUtils（数据清洗工具） | common.client.QrCodeClient；common.service.OAuthUserService；common.dto.ThirdConfigBaseDTO；common.utils.HttpClientUtils + DataHandleUtils；common.request 下 8 种 ThirdConfigRequest | common 中无独立配置，都是被各子域继承或复用 |

---

## 三、核心业务规则（通用集成铁律 + 各子域关键点）

| 规则 ID | 规则描述（if/then） | 例外 |
|---|---|---|
| R1 | **Token 本地缓存 + 自动刷新**：所有第三方平台的 access_token（钉钉/飞书/企微/MaxKB/DataEase/QCC 等）if (未过期) → 走内存 + Redis 缓存；过期前 1 分钟 自动刷新；刷新失败重试 3 次 → 打告警日志并继续用旧 token（不中断业务） | 手动测试时可加参数 `forceRefresh=true` 临时跳过缓存 |
| R2 | **第三方超时 / 失败不阻塞主业务**：if (通知/部门同步/查企查查 失败) → 失败记录日志 + 站内信给管理员；主业务（例如"线索分配完成"）**本身成功返回**；用户看到"分配成功，IM 推送稍后重试" | 关键交易流程（SSO 登录、审批回调）除外：失败直接报错，防止登录成功但权限无或审批乱序 |
| R3 | **第三方配置开关**：XXXThirdConfigRequest → if (enabled=false) → 该子域所有接口短路返回"平台未启用"；前端配置页显灰；防止调用未配置平台导致空指针或错误 | 超管临时开关测试接口可以单独传 `ignoreEnabled=true`（需鉴权） |
| R4 | **外部 URL / API 路径不写死在代码里**：所有子包下 `constant/XXXApiPaths.java` 存常量路径（如 `/v1/applications`、`/v1/workflow/run`）；baseUrl 从配置中取；两者拼接成真实 URL | 发布新版第三方时只改 constant + 配置，不动 Service 逻辑 |
| R5 | **统一响应结构封装**：所有第三方响应 → `dto/response/XXXBaseResponse` / `XXXResponseEntity` 统一包装 code+msg+data；任何非 200 / 非 SUCCESS code 都抛 GenericException 给上层统一处理 | — |
| R6 | **跨组织配置隔离**：第三方配置按 org_id 保存（或 OrganizationContext）；每个组织独立一套 appKey/secret；查询时自动拼 org_id 条件，避免串数据 | — |
| R7 | **SSO 登录 state 一次性**：OAuth State（OAuthStateService）→ 每个 state 只能用一次，有效期 10 分钟；用过即删 + 过期即删，防止 CSRF 重放攻击 | — |
| R8 | **消息推送限流**：同一用户 1 分钟内收到 IM 消息（钉钉/飞书/企微）超过 50 条 → 限流合并，改为"您有 N 条 CRM 待处理消息，请登录系统查看" | 管理员紧急通知可跳过限流（系统公告场景） |
| R9 | **部门同步只增不硬删**：第三方（钉钉/飞书/企微）的部门/用户删除 → 同步到 Cordys 时默认 =**逻辑停用/标记已删除**，不做物理 DELETE；防止误删导致历史数据（线索 owner/协作人）关联失效 | 超管有"同步清理半年前停用"脚本 |
| R10 | **AI Agent 脚本执行上下文隔离**：AgentScript 执行 MaxKB 工作流 → 每次请求独立 traceId 日志 + 请求参数脱敏记录响应（不存 prompt/token 原文）；Token 不落日志明文 | — |

---

## 四、状态/枚举/常量/配置（定位入口，易变项核对代码）

| 语义 | 枚举/类名入口 | 代码路径 |
|---|---|---|
| 第三方 OAuth 状态流转 | OAuthStateFlow 枚举（integration/sso） | integration.sso.constants.OAuthStateFlow |
| 第三方通用配置请求类型 8 种 | integration.common.request 下：De/DingTalk/Lark/MaxKB/Qcc/Tender/WeCom + ThirdConfigBaseDTO | integration.common.request.* |
| 各平台 API 路径常量 | integration.子包.constant 下：`{Xxx}ApiPaths.java`（Lark/DingTalk/WeCom/Qcc/MaxKB/Tender） | integration.*.constant.*ApiPaths.java |
| 第三方开关配置 OrganizationConfig | 部分平台级开关结合 OrganizationConfig（各平台是否启用部门同步、消息通知等） | system.domain.OrganizationConfig + OrganizationConfigConstants |
| 通知渠道配置（站内/邮件/IM） | NotificationConstants + 用户通知设置；最终由 CommonNoticeSendService 路由到 integration.*.NoticeSender | system.constants.NotificationConstants + system.notice.* |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **system（通知 Notice / 组织配置 / 调度 Schedule / 部门用户）** | integration ↔ system | ① system.notice 调各 integration NoticeSender；② ThirdDepartmentService 把外部部门同步到 system.Department + User；③ ScheduleController 手动触发"同步部门"任务；④ OrganizationConfig 存各平台开关 |
| **所有业务模块（线索/客户/商机/合同/订单/审批/follow）** | 业务模块 → integration（间接，经 system.notice） | 业务模块"动作完成"→ 发通知 CommonNoticeSendService → 根据配置调钉钉/飞书/企微 NoticeSender；审批/客户/线索公海/商机 均有大量通知 |
| **business（客户-企查查）** | customer 详情 → qcc 企业查询 | （如有）客户详情页"从企查查补全企业信息"按钮 → 后端调 QccClient.detail → 回填 CustomerField（需授权确认） |
| **business（商机/合同/订单-标讯）** | tender ↔ opportunity | （如有）标讯一键生成商机/线索 → TenderController.detail → 预填 OpportunityAddRequest（由前端拼） |
| **SSO ↔ framework.security** | sso → security | SSOService 认证成功后 → SessionUtils 登录本地用户（Shiro Session 写入 Redis）→ 重定向前端首页 |
| **DataEase ↔ system + dashboard** | dataease ← dashboard | Dashboard 配置中可指定"DataEase 嵌入式报表"→ 前端通过 DataEase SyncService 预先同步权限 → 免密嵌入（DeTempResourceDTO 临时授权） |
| **MaxKB Agent ↔ MCP / 前端 AI 聊天** | agent → AI Chat UI + MCP | 前端 AI 聊天/MCP 调用 → 后端 AgentBaseService 调 MaxKB App/Workflow API → 流式或同步返回 |

---

## 六、常见边界场景 & 处理方式（集成最容易踩坑的点）

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **第三方 IP 白名单 / 网络不通（客户内网部署）** | 所有 HTTP 客户端有 connectTimeout=10s / readTimeout=30s（可配置）；请求失败时"网络错误"友好提示 + 诊断日志中输出实际访问 baseUrl + proxy 配置 | HttpClientUtils / CSHttpClient 超时 + 代理配置 |
| **钉钉/飞书/企微 IP 变了导致回调验签失败** | 验签时不要依赖 caller IP，用官方 SDK 签名校验；回调接口按官方规范走（如有）；部署文档写明"回调 URL 必须公网可访问 + HTTPS" | NoticeSender / 回调实现（核对实现） |
| **MaxKB API 限流（QPS）** | AgentBaseService 调用前加 RateLimiter（Guava 令牌桶，默认 5QPS/key）；限流器触发时前端提示"当前排队 N 人，请稍后重试" | AgentBaseService 限流逻辑（如已有实现） |
| **SSO 回调失败（state 过期）** | 用户停留 SSO 页面超过 10 分钟再点授权 → state 已过期 → 友好提示"登录已超时，请重新发起登录"→ 自动跳回登录页 | SSOController.callback + OAuthStateService 校验 state |
| **企业/标讯查询额度耗尽（QCC 按次付费）** | QccClient 响应 code=4029（频次限制）或 429 → 返回"查询次数已用完，请联系管理员充值"；加 24h 滑动窗口避免重复请求浪费额度 | QccClient 通用响应拦截（qcc.response.QccBaseResponse 判断错误码） |
| **DataEase 同步权限时失败（DataEase 端先改了密码）** | DataEaseClient 每次先验证 Token 有效性 → 失效则自动重新登录获取新 Token（见 R1）；登录失败 3 次打告警日志 | DataEaseClient 登录失败重试 |
| **企微/飞书/钉钉消息被拒收（用户离职 / 不在群）** | NoticeSender 调用失败时回退"站内信保底送达"；同时记录失败用户 id，管理员后台查看"推送失败名单"手动修复（解除离职或重新同步） | CommonNoticeSendService 回退站内信逻辑 |
| **二维码生成后用户迟迟不扫码** | QrCodeClient 生成二维码 + SSO state 绑定；过期（10 分钟）后清理；前端轮询显示"二维码已过期，请刷新" | SSO state TTL + 前端轮询接口 |
