---
type: application
status: official
evidence: backend/pom.xml + backend/app + backend/crm + backend/framework 代码结构
owner: cordys-crm
verify-required: true
verify-note: 实际 Controller/Service/Mapper 结构需回到代码核对
updated: 2026-08-11
---

# Cordys CRM · 后端应用总览

> 本文件是后端（Java 21 + Spring Boot）的应用级总览。
> 应用代码路径：`backend/`（Maven 多模块）

---

## 一、应用职责

**负责什么**：
- 提供 CRM 业务全部后端 API（PC 端、移动端、MCP、第三方集成）
- 业务逻辑处理与持久化（MySQL）
- 认证授权（Shiro + Session Redis）
- 定时任务调度（Quartz）
- 文件存储（本地/S3）
- 消息通知（站内信/邮件/企微/钉钉/飞书）
- 数据导出（Excel）
- 第三方集成（MaxKB/DataEase/企查查/钉钉/飞书/企微/SSO/招投标）

**不负责什么**：
- 不负责页面渲染（前端负责）
- 不负责直接执行 AI 模型推理（MCP + Cockpit 负责）
- 不负责 BI 报表渲染（DataEase 负责，后端仅提供数据接口）

---

## 二、Maven 模块结构

```
backend/                          ← Maven 聚合模块
├── framework/                    ← 通用基础框架
│   └── 包: cn.cordys.{aspectj,common,config,context,
│          excel,file,mybatis,registry,security}
├── crm/                          ← CRM 业务模块（核心代码量最大）
│   └── 包: cn.cordys.{common,config,crm.{approval,clue,
│          contract,customer,dashboard,follow,form,home,
│          integration,opportunity,order,product,search,system}}
└── app/                          ← Spring Boot 启动模块
    └── 包: cn.cordys.Application + listener

依赖方向: app → crm → framework（单向，严禁反向依赖）
```

模块 POM 路径：
- 根: [backend/pom.xml](file:///e:/工作/金数湾/CordysCRM/backend/pom.xml)
- framework: [backend/framework/pom.xml](file:///e:/工作/金数湾/CordysCRM/backend/framework/pom.xml)
- crm: [backend/crm/pom.xml](file:///e:/工作/金数湾/CordysCRM/backend/crm/pom.xml)
- app: [backend/app/pom.xml](file:///e:/工作/金数湾/CordysCRM/backend/app/pom.xml)

---

## 三、上下游

**上游（调用方）**：
| 调用方 | 协议 | 鉴权方式 | 主要接口 |
|---|---|---|---|
| PC 前端 (web) | HTTP REST + SSE | Shiro Session | /api/* |
| 移动端 (mobile) | HTTP REST | Shiro Session | /api/* |
| MCP Server (8082) | HTTP REST | API Key 或内部调用 | /api/agent/* |
| 第三方系统 | HTTP REST | API Key | /api/*（按权限节点） |
| SSO OAuth 回调方 | HTTP redirect | OAuth State | /login/oauth2/code/* |

**下游（被调用方）**：
| 下游 | 协议 | 用途 | 配置入口 |
|---|---|---|---|
| MySQL 8 | JDBC (HikariCP) | 业务持久化 + Quartz | spring.datasource.* |
| Redis | Redisson Client | Session + 缓存 + 发布订阅 | spring.data.redis.* |
| MaxKB | HTTP REST | AI 知识库问答 | integration/agent 配置 |
| DataEase | HTTP REST | BI 嵌入式数据 | integration/dataease 配置 |
| 企查查 | HTTP REST | 企业工商信息查询 | integration/qcc 配置 |
| 钉钉/飞书/企微 | HTTP Webhook | 消息通知 + 组织同步 | integration/{dingtalk,lark,wecom} |
| SMTP 邮件服务器 | SMTP | 邮件发送 | spring.mail.* |
| Cockpit (8088) | HTTP REST | AI 复杂能力调度 | cockpit.base.url |

---

## 四、核心模块入口速查

| 业务模块 | 顶层包路径 | 代码核对 |
|---|---|---|
| 审批流 approval | `cn.cordys.crm.approval` | `controller/` + `service/` 6 个 + `domain/` |
| 线索 clue | `cn.cordys.crm.clue` | 8 Controller + 9 Service |
| 客户 customer | `cn.cordys.crm.customer` | 10+ Service（客户/联系人/公海/协作/关系） |
| 商机 opportunity | `cn.cordys.crm.opportunity` | 8 Controller + 12 Service（商机/报价/阶段/规则） |
| 合同 contract | `cn.cordys.crm.contract` | 7 Service（合同/发票/收款/工商抬头） |
| 订单 order | `cn.cordys.crm.order` | 3 Controller + 6 Service |
| 产品 product | `cn.cordys.crm.product` | 2 Controller + 10+ Service（产品/价格/字段） |
| 跟进 follow | `cn.cordys.crm.follow` | BaseFollowUpService + 9 Service（计划/记录/评论） |
| 自定义表单 form | `cn.cordys.crm.form` | 3 Controller + 7 Service |
| 仪表盘 dashboard | `cn.cordys.crm.dashboard` | 3 Service |
| 全局搜索 search | `cn.cordys.crm.search` | GlobalSearch + AdvancedSearch Controller |
| 系统管理 system | `cn.cordys.crm.system` | 26 Controller / 40+ Service（用户/角色/部门/...） |
| 第三方集成 integration | `cn.cordys.crm.integration` | agent/dataease/dingtalk/lark/wecom/qcc/sso/tender |

---

## 五、启动与初始化流程

**启动类**：`cn.cordys.Application` → [Application.java](file:///e:/工作/金数湾/CordysCRM/backend/app/src/main/java/cn/cordys/Application.java)

**AppListener 启动初始化顺序**（ApplicationRunner）：
1. 设置 `SessionUser.secret`（AES 密钥）
2. 初始化 `SqlInjectionChecker` 危险模式
3. 初始化 `DefaultUidGenerator`（Snowflake UID）
4. 初始化 RSA 配置（从 Redis 读取或首次生成）
5. 启动 Quartz 定时任务（ExtScheduleService）
6. 打印 HikariCP 连接池状态
7. 初始化默认组织数据（DataInitService.initOneTime，仅首次）
8. 停止遗留导出任务（异常中断的导出）
9. 清理表单缓存

---

## 六、请求处理全链路

```
HTTP Request
  │
  ▼
ShiroFilter 过滤链 (cn.cordys.crm.common.security)
  ├─ ApiKeyFilter       → API Key 鉴权（第三方系统）
  ├─ CsrfFilter         → CSRF Token + Referer 校验（浏览器请求）
  ├─ AuthFilter         → Shiro Session 鉴权（未认证返回 401）
  └─ FileAccessAuthFilter → 附件预览 Cookie 认证
  │
  ▼
Controller
  ├─ @CsPermission        → 功能权限切面校验
  ├─ @HitApproval         → 审批节点判断（审批中限制操作）
  ├─ @OperationLog        → 操作日志 AOP（异步落库）
  └─ @SendNotice          → 消息通知触发
  │
  ▼
Service
  ├─ BaseService           → ID→名称映射、创建/更新/责任人回填
  ├─ DataScopeService      → 数据权限范围拼接
  ├─ PermissionUtils       → 权限判断
  └─ @Transactional        → 事务边界（默认回滚 RuntimeException）
  │
  ▼
Mapper (BaseMapper<E> + LambdaQueryWrapper)
  ├─ MybatisInterceptor    → 敏感字段 AES 加解密
  └─ 组织隔离 + 逻辑删除    → 自动拼接 WHERE organization_id=? AND deleted=0
  │
  ▼
MySQL (HikariCP 连接池)
  │
  ▼
ResultHolder (统一响应包装 + ResultResponseBodyAdvice)
  │
  ▼
HTTP Response (JSON)
```

---

## 七、前后端接口约定

- **Base URL**：`http://host:8081/api/`
- **鉴权方式（浏览器）**：Cookie: `JSESSIONID`（Redis Session）+ Header: `X-CSRF-Token`
- **鉴权方式（API Key）**：Header: `X-API-Key: {key}`
- **统一响应结构**：
  ```json
  {
    "code": 100200,        // 100200=SUCCESS, 100500=FAILED, 100401=UNAUTHORIZED
    "message": "success",
    "data": { /* ... */ },
    "success": true
  }
  ```
- **分页参数**：
  ```
  GET /api/clue/list?pageNum=1&pageSize=20
  Response: { list: [], total: 0, pageNum: 1, pageSize: 20 }
  ```
- **错误码定义**：`cn.cordys.framework.common.response.enums.CrmHttpResultCode`

---

## 八、AI 进入后端的推荐读取路径

```
第1步: 读本文件 application-backend.md
第2步: 读 domain/product/ → 了解通用产品能力（主干流程）
第3步: 读 domain/solution/ → 了解差异化逻辑（如有）
第4步: 读 domain/base/api-{module}.md → 定位 Controller/Service 入口
第5步: 读 domain/base/repository-{module}.md → 定位 Mapper/Domain/表
第6步: 必要时读 tech/ → 了解研发规范与常见踩坑
第7步: 回到代码仓库，实际读取 Controller → Service → Mapper 源码
```
