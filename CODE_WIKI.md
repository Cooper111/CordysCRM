# Cordys CRM Code Wiki

> 本文档为 Cordys CRM 项目仓库的结构化代码 Wiki,涵盖项目整体架构、模块职责、关键类与函数、依赖关系以及运行方式。
>
> - 仓库根目录:`E:\工作\金数湾\CordysCRM`
> - 上游开源:`https://github.com/1Panel-dev/CordysCRM`
> - 出品方:飞致云(FIT2CLOUD)
> - 当前版本:`main`(对应 `${revision}` 占位,版本随 release tag 演进)

---

## 目录

- [1. 项目概览](#1-项目概览)
- [2. 整体架构](#2-整体架构)
- [3. 技术栈](#3-技术栈)
- [4. 仓库目录结构](#4-仓库目录结构)
- [5. 后端架构与模块职责](#5-后端架构与模块职责)
- [6. 前端架构与模块职责](#6-前端架构与模块职责)
- [7. 资源文件与数据库迁移](#7-资源文件与数据库迁移)
- [8. 依赖关系](#8-依赖关系)
- [9. 项目运行方式](#9-项目运行方式)
- [10. CI/CD 与工程规范](#10-cicd-与工程规范)
- [11. 关键设计总结](#11-关键设计总结)

---

## 1. 项目概览

**Cordys CRM** 是新一代开源 AI CRM(客户关系管理)系统,由飞致云匠心出品,集信息化、数字化、智能化于一体。其核心特点:

- **灵活配置 · 高效协同**:现代化架构,精细权限与模块化配置,无缝集成主流办公平台。
- **安全自主 · 深度可控**:为私有化部署而生,数据 100% 自主可控;开放 API 与标准接口。
- **智能 BI · 决策赋能**:深度融合 DataEase 分析引擎,销售数据可视化呈现。
- **AI 赋能 · 智能提效**:开放 CRM Skills 接口,智能助手(OpenClaw、WorkBuddy 等)7×24 在线,从线索筛选到成单分析全流程提效。

业务覆盖:线索 → 客户 → 联系人 → 商机 → 报价 → 合同 → 发票 → 收款 → 订单 → 产品 → 仪表盘 → 审批流 → 智能体 → 第三方集成(钉钉/飞书/企微/企查查/DataEase/SSO/招投标)。

---

## 2. 整体架构

Cordys CRM 采用 **前后端分离 + 单体多模块** 架构,通过 Docker 一键化部署。

```
┌─────────────────────────────────────────────────────────────────────┐
│                         浏览器 / 移动端                              │
│  ┌──────────────────────────┐    ┌──────────────────────────┐      │
│  │  web (PC, Naive UI)      │    │  mobile (Vant UI)        │      │
│  └─────────────┬────────────┘    └────────────┬─────────────┘      │
└────────────────┼──────────────────────────────┼────────────────────┘
                 │  HTTP / SSE / OAuth          │
                 ▼                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│              Spring Boot 应用 (端口 8081)                            │
│  ┌──────────┐  ┌──────────┐  ┌────────────────────────────────┐   │
│  │  app     │  │ crm      │  │  framework                     │   │
│  │ (启动+路由)│->│ (业务)   │->│  (通用能力: AOP/MyBatis/安全)   │   │
│  └──────────┘  └──────────┘  └────────────────────────────────┘   │
│        │              │                                            │
│        │              ▼                                            │
│        │     Shiro(AuthFilter/CsrfFilter/ApiKeyFilter)             │
│        │     MyBatis + PageHelper + Flyway                         │
│        ▼                                                           │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐    │
│  │  MySQL     │  │  Redis     │  │  Quartz    │  │ Flyway   │    │
│  │ (业务/调度) │  │ (Session/  │  │  (定时任务) │  │ (迁移)   │    │
│  │            │  │  缓存/订阅) │  │            │  │          │    │
│  └────────────┘  └────────────┘  └────────────┘  └──────────┘    │
└─────────────────────────────────────────────────────────────────────┘
        ▲                                  ▲
        │                                  │
┌──────────────────────┐         ┌──────────────────────┐
│  MCP Server (8082)   │         │  Cockpit (8088)      │
│  AI 智能创建/录入/查重│         │  智能服务后端          │
└──────────────────────┘         └──────────────────────┘
        ▲                                  ▲
        │  外部 AI 助手(WorkBuddy/OpenClaw 等)通过 MCP 调用 CRM Skills
        ▼
┌────────────────────────────────────────────────────────────────────┐
│  外部集成: DataEase(BI) / MaxKB(AI 知识库) / 钉钉 / 飞书 / 企微 /   │
│            企查查 / SSO / 招投标                                    │
└────────────────────────────────────────────────────────────────────┘
```

**关键架构特点**:

1. **多模块 Maven 工程**:`cordys-crm`(root)→ `backend` + `frontend`;`backend` → `framework` + `crm` + `app`。
2. **前后端同包发布**:`app` 模块通过 `maven-antrun-plugin` 在 `generate-resources` 阶段将前端 `dist` 复制到 `src/main/resources/static`,最终随 Spring Boot Fat JAR 一同发布。
3. **All-in-One 部署**:容器内置 MySQL / Redis / MCP Server / Cockpit,通过 `cordys-crm.properties` 中 `*.embedded.enabled` 开关控制是否启用内置服务,支持切换到外部服务。
4. **Java 21 + 虚拟线程**:`spring.threads.virtual.enabled=true`,提升 IO 密集型吞吐。
5. **Shiro + Session-Redis**:基于 Shiro 做认证授权,Session 走 Redis 索引式存储,支持多实例水平扩展。

---

## 3. 技术栈

### 后端

| 分类 | 技术 / 版本 |
|---|---|
| 语言 / 运行时 | Java 21 |
| 应用框架 | Spring Boot 3.5.14(基于 Jetty,排除 Tomcat) |
| 安全框架 | Apache Shiro 2.1.0(Jakarta classifier) |
| ORM | MyBatis Spring Boot Starter 3.0.5 + PageHelper 6.1.1 |
| 数据库 | MySQL(flyway-core + flyway-mysql 管理迁移) |
| 缓存 / Session | Redis(Redisson 3.52.0 + spring-session-data-redis) |
| 调度 | Quartz(`cn.cordys:quartz-spring-boot-starter` 1.0.1) |
| API 文档 | springdoc-openapi-starter-webmvc-ui 2.8.16 |
| Excel | FastExcel 1.3.0 |
| JWT | com.auth0:java-jwt 3.12.1 |
| 邮件 | org.eclipse.angus:jakarta.mail 2.0.5 |
| 通用工具 | commons-lang3 / commons-collections4 / commons-text / commons-io / commons-compress / commons-codec |
| 持久化 API | jakarta.persistence-api 3.2.0 |
| 测试 | spring-boot-starter-test + Testcontainers 2.0.3(embedded-mysql / embedded-redis) |
| 构建 | Maven + flatten-maven-plugin(CI Friendly `${revision}`)+ JaCoCo 0.8.12 |

### 前端

| 分类 | 技术 / 版本 |
|---|---|
| 框架 | Vue 3.5.22 + TypeScript 5.9.3 |
| 构建工具 | Vite + @vitejs/plugin-legacy 6.0(兼容非 IE11) |
| 状态管理 | Pinia 2.3 + pinia-plugin-persistedstate |
| 路由 | Vue Router 4.5(hash 模式) |
| 国际化 | vue-i18n 9.13 |
| PC 端 UI | Naive UI |
| 移动端 UI | Vant UI(配 `postcss-pxtorem` rem 适配) |
| HTTP | axios 1.7 |
| 图表 | echarts 6.1 + vue-echarts |
| AI 聊天 | @ai-sdk/vue + ai(markdown-it + katex + highlight.js + dompurify) |
| 流式画布 | AntV X6(crm-flow 审批流画布) |
| 工具库 | @vueuse/core、lodash-es、dayjs、jsencrypt、jspdf、html2canvas-pro、localforage、mitt、query-string |
| 包管理 | pnpm 10.4.1(monorepo + workspace) |
| 代码规范 | ESLint 9 + Prettier + Stylelint + Husky + commitlint + lint-staged |
| 样式 | Tailwind CSS 3.4 + Less |

### 中间件 / 基础设施

| 分类 | 技术 |
|---|---|
| 数据库 | MySQL 8 |
| 缓存 | Redis |
| 容器 | Docker(多阶段构建,基础镜像 `ghcr.io/cordys-dev/cordys-base:latest`) |

---

## 4. 仓库目录结构

```
CordysCRM/
├── .github/                     # GitHub 配置(Issue/PR 模板、CI workflows)
├── backend/                     # 后端 Maven 多模块工程
│   ├── app/                     # 启动模块(入口、监听器、配置)
│   ├── framework/               # 基础框架模块(通用能力)
│   ├── crm/                     # CRM 业务模块
│   └── pom.xml
├── frontend/                    # 前端 pnpm monorepo
│   ├── packages/
│   │   ├── lib-shared/          # @lib/shared 共享库
│   │   ├── web/                 # @cordys/web PC 端
│   │   └── mobile/              # @cordys/mobile 移动端
│   ├── .husky/
│   ├── package.json
│   └── pnpm-workspace.yaml
├── installer/                   # 安装部署相关
│   ├── Dockerfile
│   ├── conf/
│   ├── shells/
│   ├── mcp/
│   └── cockpit/
├── mvnw / mvnw.cmd
├── pom.xml
├── BUILD.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── LICENSE
├── OWNERS
└── lombok.config
```

---

## 5. 后端架构与模块职责

后端采用 **三层多模块** 分层架构:

```
framework  (通用能力: AOP / 通用响应 / MyBatis 增强 / 安全 / 文件 / UID)
    ▲
    │ depends on
    │
   crm   (业务公共层 + 业务子模块群)
    ▲
    │ depends on
    │
   app   (Spring Boot 启动 + 全局配置 + 路由)
```

所有 Java 代码统一使用包前缀 `cn.cordys.*`。

### 5.1 app 启动模块

| 文件 | 职责 |
|---|---|
| `Application.java` | Spring Boot 启动类。`@SpringBootApplication`(排除 `QuartzAutoConfiguration` / `LdapAutoConfiguration` / `Neo4jAutoConfiguration`),`@PropertySource` 加载 `classpath:commons.properties` 与 `file:/opt/cordys/conf/cordys-crm.properties`,`@ServletComponentScan` |
| `listener/AppListener.java` | `ApplicationRunner` 启动初始化:设置 `SessionUser.secret`、初始化 `SqlInjectionChecker` 危险模式、初始化 `DefaultUidGenerator`、初始化 RSA 配置(从 Redis 读取或生成)、启动 Quartz 定时任务(`ExtScheduleService`)、打印 HikariCP 状态、初始化默认组织数据(`DataInitService.initOneTime`)、停止遗留导出任务、清理表单缓存 |
| `listener/Index.java` | `@Controller` 路由:`/web` → index.html、`/mobile` → mobile/index.html、`/login` → index.html |
| `listener/CustomError.java` | 全局错误控制器,`/error` → 重定向到 `/` |
| `commons.properties` | 应用主配置:端口 8081、HikariCP 连接池、Quartz、MyBatis、Flyway、文件上传、Redisson Session、Swagger、i18n、白名单、cockpit 等基础设置 |

### 5.2 framework 基础框架模块

子包结构:

```
framework/src/main/java/cn/cordys/
├── aspectj/      # AOP 操作日志
├── common/       # 统一响应 / 异常 / 分页 / UID / 工具
├── config/       # 框架级配置
├── context/      # 组织上下文
├── excel/        # EasyExcel 导出
├── file/engine/  # 文件存储引擎
├── mybatis/      # 通用 Mapper + Lambda 查询 + 拦截器
├── registry/     # 导出线程注册表
└── security/     # Shiro 过滤器 / Session 工具
```

#### 5.2.1 aspectj(操作日志切面)

| 类 / 接口 | 职责 |
|---|---|
| `annotation/OperationLog` | 操作日志注解,`@Repeatable`,属性:operator/type/module/resourceId/resourceName/success/fail/extra,支持 SpEL 表达式 |
| `aop/OperationLogAopAdvisor` | 切面 Advisor,匹配 `cn.cordys.*.service` 包下带注解方法 |
| `builder/parse/OperationLogExpressionEvaluator` | SpEL 表达式求值 |
| `context/OperationLogContext` | ThreadLocal 上下文(resourceId/resourceName/originalValue/modifiedValue) |
| `handler/OperationLogHandler` | **接口** `void handleLog(LogDTO)` |
| `handler/OperationLogService` | `@Service`,补全模块字段、请求信息,异步调用 handler |

#### 5.2.2 common(统一响应 / 异常 / 分页 / UID / 工具)

**统一响应**:
- `ResultHolder`(code/message/messageDetail/data,success()/error())+ `ResultResponseBodyAdvice`(自动包装)+ `RestControllerExceptionHandler`(全局异常)+ `CrmHttpResultCode`(SUCCESS=100200 / FAILED=100500 / UNAUTHORIZED=100401 / FORBIDDEN=100403 / VALIDATE_FAILED=100400 / NOT_FOUND=100404)

**异常**:
- `IResultCode`(接口)+ `GenericException`(通用运行时异常,贯穿全栈)

**分页**:
- `Pager<T>` + `PageUtils` + `PagerWithOption`

**UID(Snowflake)**:
- `DefaultUidGenerator`(64bit:sign(1)+time(30)+worker(21)+seq(12),epoch=2025-01-01)+ `WorkerIdAssigner`(数据库分配)+ `RingBuffer`(环形缓冲预生成)

**工具类**:
- `EncryptUtils`(AES+MD5)/ `RsaUtils`(RSA)/ `BeanUtils` / `JSON` / `Translator` / `ServletUtils` / `TimeUtils` / `JsonDifferenceUtils` / `KMeansUtils`/ `LogisticRegressionUtils` / `SqlInjectionChecker` / `CommonBeanFactory`

#### 5.2.3 config(框架级)

- `DataAccessConfig`(数据访问层)
- `I18nConfig`(国际化 MessageSource)
- `MybatisInterceptorConfig`(加解密拦截器,默认 `EncryptUtils.aesEncrypt/aesDecrypt`)
- `RequestParamTrimConfig`(请求参数去空格过滤)

#### 5.2.4 file(文件存储引擎)

- `FileRepository`(核心接口:saveFile/delete/getFile 等)
- `FileCenter`(静态工厂,根据 `StorageType.LOCAL/S3` 路由)
- `LocalRepository`(本地文件系统)+ `S3Repository`(S3/MinIO)

#### 5.2.5 mybatis(通用 Mapper + Lambda 查询 + 拦截器)

- `BaseMapper<E>`(核心通用 Mapper 接口:insert/batchInsert/updateById/delete/selectByLambda/count/upsert/exist 等,内置 SqlProvider)
- `MybatisInterceptor`(拦截 update/query,按 `MybatisInterceptorConfig` 对敏感字段加解密)
- `LambdaQueryWrapper<T>`(类型安全的 Lambda 条件构造,内置 SQL 注入检测)

#### 5.2.6 security(Session 工具)

- `ShiroFilter`(基础过滤器链)
- `SessionUser`(用户对象,含 `secret` 静态密钥)
- `SessionUtils`(getUserId/getSessionId/putUser)
- `FileAccessAuthFilter`(附件预览 Cookie 认证)

### 5.3 crm 业务模块

```
crm/src/main/java/cn/cordys/
├── common/      # 业务公共层(权限 / 调度 / Redis / 字段解析器 / BaseService)
├── config/      # 业务配置(Shiro / Redis / Mybatis / Schedule / Session / Async)
└── crm/         # 业务子模块群
    ├── approval/     # 审批流
    ├── clue/         # 线索 + 线索池
    ├── contract/     # 合同 + 发票 + 付款
    ├── customer/     # 客户 + 联系人 + 公海
    ├── dashboard/    # 仪表盘
    ├── follow/       # 跟进计划 / 记录 + 评论
    ├── form/         # 自定义表单
    ├── home/         # 首页统计
    ├── integration/  # 第三方集成(agent / dataease / dingtalk / lark / qcc / sso / wecom / tender)
    ├── opportunity/  # 商机 + 报价 + 阶段
    ├── order/        # 订单 + 阶段
    ├── product/      # 产品 + 价格
    ├── search/       # 全局 / 高级搜索
    └── system/       # 系统管理
```

#### 5.3.1 crm/common 业务公共层

**security(Shiro Realm 与过滤器)**:
- `LocalRealm`(Shiro Realm,LOCAL 模式,委托 `UserLoginService.authenticateUser`)
- `AuthFilter`(未认证返回 401)、`ApiKeyFilter`(API Key 认证)、`CsrfFilter`(CSRF token + Referer 校验)
- `ApiKeyHandler`(API Key 识别)、`SSRFValidator`(SSRF 防护)

**service(Base 类)**:
- `BaseService`(核心基类,ID→名称映射、创建/更新/责任人回填、操作日志上下文写入、审批节点判断)
- `BaseExportService` / `ExportExecutor` / `DataInitService` / `DataScopeService` / `FieldSourceServiceProvider`

**permission(权限)**:
- `Permission`(接口)、`@CsPermission` / `@CsBatchPermission`(自定义注解)
- `CsPermissionAspect`(切面校验)、`PermissionUtils.hasPermission()` / `PermissionCache`

**其他公共组件**:
- `schedule/`(BaseScheduleJob/ScheduleManager/ScheduleService-Quartz 封装)
- `redis/`(MessagePublisher/MessageSubscriber/TopicConsumer-Redis 发布订阅)
- `resolver/field/`(CheckBox/DateTime/Department/Formula/Industry/Location/Member/Number/Phone/Picture/Radio/Select/Text 等 18+ 字段值解析器)

#### 5.3.2 crm/config 业务配置类

- `ShiroConfig`(核心:ShiroFilterFactoryBean 四过滤器 + DefaultWebSecurityManager + LocalRealm + AuthorizationAttributeSourceAdvisor)
- `RedisConfig`(@EnableCaching + RedisCacheManager TTL 1h + RedisTemplate + Redis 发布订阅容器)
- `MybatisConfig` / `SessionConfig` / `ScheduleConfig` / `AsyncConfig`

#### 5.3.3 crm/crm 业务子模块摘要

| 模块 | 核心 domain / service | 功能 |
|---|---|---|
| **approval** | `ApprovalFlow/Instance/Node/Task/Record`;`@HitApproval` 注解 + `HitApprovalAspect`;6 个 Service | 审批流:节点类型/审批人类型/加签/驳回,自动触发 |
| **clue** | `Clue/ClueField/CluePool/ClueOwner/ClueCapacity`;8 个 Controller + 9 个 Service | 线索 / 线索池 / 分配 / 容量 / 转客户 / 导出 |
| **customer** | `Customer/CustomerField/CustomerContact/CustomerPool/CustomerCapacity/CustomerCollaboration/CustomerRelation`;10+ Service | 客户 / 联系人 / 公海 / 协作 / 关系图 / 容量 |
| **opportunity** | `Opportunity/OpportunityQuotation/OpportunityStageConfig/OpportunityRule`;8 个 Controller + 12+ Service | 商机 / 报价 / 阶段 / 规则 / 视图 / 跟进 |
| **contract** | `Contract/ContractInvoice/ContractPaymentPlan/ContractPaymentRecord/BusinessTitle`;7 个 Service | 合同 / 发票 / 收款计划 / 收款记录 / 工商抬头 |
| **order** | `Order/OrderStageConfig/OrderSnapshot`;3 个 Controller + 6 个 Service | 订单 / 订单阶段 / 视图 / 导出 |
| **product** | `Product/ProductPrice/ProductField/ProductPriceField`;2 个 Controller + 10+ Service | 产品 / 价格 / 字段 / 导出 / 日志 |
| **follow** | `FollowUpPlan/FollowUpRecord/Comment`;`BaseFollowUpService` + 9 个 Service | 跟进计划 / 跟进记录 / 评论 / 字段 / 日志 |
| **form** | `CustomForm/CustomFormData/CustomFormRole`;3 个 Controller + 7 个 Service | 自定义表单 / 数据 / 角色 / 导出 |
| **dashboard** | `Dashboard/DashboardCollection/DashboardModule`;3 个 Service | 仪表盘 / 收藏 / 模块 / 排序 |
| **search** | `GlobalSearchController` / `AdvancedSearchController`;`*SearchServiceFactory` 工厂 | 全局搜索 / 高级搜索 / 用户搜索配置 / 字段脱敏 |
| **system** | 26 个 Controller、35+ domain、40+ Service;含 notice(job/sender/sse)子系统、job/listener、excel 导入导出、dto/field 类型体系(20+ Field) | 用户 / 角色 / 部门 / 字典 / 模块 / 导航 / 公告 / 附件 / 通知 / 日志 / License / 导出任务中心 / 个人中心 / 国际化 / 组织 / 智能体配置 |
| **integration** | agent(MaxKB)/dataease(BI)/dingtalk/lark/wecom(通知+组织)/qcc(企查查)/sso(OAuth)/sync/tender + common(HttpClientUtils/QrCodeClient) | 智能体 / BI / 钉钉 / 飞书 / 企微 / 企查查 / SSO / 招投标 / 组织同步 |

### 5.4 后端关键类与函数说明

#### 请求处理链路

```
HTTP 请求 → ShiroFilter(ApiKeyFilter → CsrfFilter → AuthFilter → FileAccessAuthFilter)
         → Controller(@CsPermission / @HitApproval / @OperationLog / @SendNotice)
         → Service(BaseService, @Transactional)
         → Mapper(BaseMapper + LambdaQueryWrapper)
         → MybatisInterceptor(字段加解密)
         → MySQL(HikariCP)
         → ResultHolder(统一响应 + 异常处理)
```

#### 关键工具函数速查

| 函数 | 所在类 | 说明 |
|---|---|---|
| `ResultHolder.success(data)` | `framework.common.response.handler.ResultHolder` | 构造统一成功响应 |
| `userMapper.selectListByLambda(new LambdaQueryWrapper<User>().eq(User::getId,"admin"))` | `BaseMapper` + `LambdaQueryWrapper` | Lambda 类型安全查询 |
| `OperationLogContext.putVariable(key, LogExtraDTO.builder()...)` | `OperationLogContext` | 注入操作日志上下文 |
| `PermissionUtils.hasPermission(permission)` | `PermissionUtils` | 权限判断 |
| `EncryptUtils.aesEncrypt(text)` | `EncryptUtils` | AES 加密(MybatisInterceptor 调用) |
| `RsaUtils.encrypt(plainText, publicKey)` | `RsaUtils` | RSA 加密(登录密码) |
| `FileCenter.getRepository(storageType)` | `FileCenter` | 获取文件存储实现 |
| `SessionUtils.getUserId()` | `SessionUtils` | 获取当前登录用户 ID |
| `DefaultUidGenerator.getUID()` | `DefaultUidGenerator` | 生成 Snowflake 分布式 UID |

---

## 6. 前端架构与模块职责

前端采用 **pnpm monorepo** 模式:`lib-shared`(公共)+ `web`(PC / Naive UI)+ `mobile`(移动端 / Vant UI)。

### 6.1 lib-shared 共享包 (@lib/shared)

```
lib-shared/
├── ai-chat/        # AI 聊天组件 + @ai-sdk/vue 运行时
├── api/
│   ├── http/       # CordysAxios 封装 + AxiosCanceler + AxiosTransform
│   ├── modules/    # 按业务模块组织 API(agent/ai/clue/customer/opportunity/contract/order/product/follow/dashboard/home/customForm/sys)
│   └── requrls/    # 与 modules 一一对应的 URL 常量
├── enums/          # httpEnum/commonEnum/moduleEnum/clueEnum/customerEnum/contractEnum/opportunityEnum/formDesignEnum/formula 等
├── hooks/useI18n.ts
├── locale/         # setupI18n / useLocale / loadLocalePool(i18n 核心)
├── method/         # 深合并/URL参数/RSA加密/SSE/树操作/格式化/加密/浏览器识别/PDF导出/拖拽等工具
├── models/         # 各业务 TS 类型(clue/customer/system 子包 + 各模块单文件)
└── types/          # axios.d.ts / global.d.ts
```

### 6.2 web PC 端包 (Naive UI)

```
web/src/
├── main.ts                  # setupApp(): store → setupI18n(阻塞)→ CrmIcon → 语言初始化 → router → localforage → directive → mount
├── App.vue                  # NConfigProvider + NMessageProvider + NDialogProvider 包裹 RouterView;OAuth 三方登录;license/主题/SSE
├── config/ (11 个)          # business / clue / contract / follow / globalTask / opportunity / process / system / workbench / pathMap
├── router/                  # hash 模式 + routes/modules 自动加载 + guard(permission + userLoginInfo)
├── store/                   # Pinia: app(index + types + visit) + user + overview + view + setting/license
├── hooks/ (30+)             # usePermission / useFormCreate* / useApproval* / usePathMap / useOpen* / useDiscreteApi / useUser / useTableStore / useLocalForage / useModal 等
├── views/ (13 个业务目录)   # agent / base / clueManagement / contract / customForm / customer / dashboard / opportunity / order / product / system / tender / workbench
├── layout/                  # default-layout + full-page-layout + page-content + header/sider
├── directive/               # v-permission + v-validateExpiration
├── components/
│   ├── business/            # ai-chat / crm-approval / crm-flow(X6) / crm-form-create / crm-formula + editor / crm-follow / crm-stage-board / crm-status-config / crm-comment / crm-task / crm-city-select 等
│   └── pure/                # crm-table / crm-chart(5种) / crm-advance-filter / crm-tree / crm-upload / crm-pagination / crm-drawer / crm-modal / crm-card / crm-tag 等
└── locale/                  # zh-CN / en-US (common + sys + settings + 组件/页面就近定义自动聚合)
```

### 6.3 mobile 移动端包 (Vant UI)

```
mobile/src/
├── main.ts                  # useLocale(showLoadingToast: Vant)
├── App.vue                  # Suspense 包裹;先 changeLocale → userStore.isLogin(true);三方浏览器走 OAuth
├── config/ (2 个)           # follow(复用 lib-shared 枚举 + API) + mine
├── router/                  # 根路径重定向 /loading;modules: clue / customer / mine / opportunity / workbench
├── store/                   # app + setting/license + user(同 web 结构)
├── hooks/ (5 个)            # useFormCreateApi / useFormCreateTransform / useHiddenTab / useLogin(mobile专用) / useUser
├── views/ (6 个业务目录)    # base / clue / customer / mine / opportunity / workbench(含 agent/ai-chat/approval/duplicateCheck/follow/task)
└── 与 web 差异:             # base 额外: AUTH_DISABLED_ROUTE / AUTH_LOGIN_LOADING_ROUTE;VantResolver 自动导入;postcss-pxtorem(rem);无 localforage;base='/mobile/'
```

**web vs mobile 业务覆盖差异**:

| 功能模块 | web | mobile |
|---|---|---|
| 线索 / 线索池 | ✓ | ✓ |
| 客户 / 联系人 / 公海 | ✓ | ✓ |
| 商机 / 报价 | ✓ | ✓(无报价) |
| 合同 / 发票 / 收款 | ✓ | ✗ |
| 订单 | ✓ | ✗ |
| 产品 / 价格 | ✓ | ✗ |
| 仪表盘 | ✓ | ✗ |
| 智能体(独立页) | ✓ | ✗ |
| 自定义表单 | ✓ | ✗ |
| 标讯 | ✓ | ✗ |
| 系统管理(组织/角色/流程/模块/日志/消息) | ✓ | ✗ |
| 工作台(含 AI 聊天/审批/查重) | ✓ | ✓ |
| 我的 | ✗(含在个人中心) | ✓ |

### 6.4 前端关键文件说明

#### 构建配置

```
packages/{web,mobile}/config/
├── vite.config.base.ts   # Vue 插件 + 别名(@ / @locale) + define + Less 全局变量
│                         # mobile 额外: VantResolver + postcss-pxtorem + base='/mobile/'
├── vite.config.dev.ts    # merge base:开发 server / proxy
├── vite.config.prod.ts   # merge base: gzip 压缩 + 包分析 + legacy 兼容
│                         # manualChunks: vue 包 / echarts 包
├── plugin/
│   ├── compress.ts       # vite-plugin-compression(gzip / brotli)
│   └── visualizer.ts     # rollup-plugin-visualizer
└── utils/index.ts
```

#### 国际化机制(i18n)

- **核心**: `lib-shared/locale/index.ts` 的 `setupI18n(app)` 从 localStorage 读 `CRM-locale`,动态 `import()` 加载语言包,`createI18n({legacy:false})`
- **切换**: `useLocale().changeLocale(_locale)` → 动态 import → 注入 i18n + dayjs → 存 localStorage → `window.location.reload()`
- **就近聚合**: 各端 `locale/{zh-CN,en-US}/index.ts` 用 `import.meta.glob('../../components/**/locale/*.ts')` 与 `import.meta.glob('../../views/**/locale/*.ts')` 自动收集所有组件 / 页面下的局部语言包,合并导出
- **UI 库同步**: Naive UI(`n-config-provider` 的 `zhCN/enUS`)/ Vant locale 跟随切换

---

## 7. 资源文件与数据库迁移

### 7.1 后端资源文件(crm/src/main/resources)

| 路径 | 用途 |
|---|---|
| `permission.json` | 权限定义树(SYSTEM/CUSTOMER/CLUE/OPPORTUNITY/CONTRACT/ORDER/PRODUCT...分组,含 READ/ADD/UPDATE/DELETE/IMPORT/SYNC 等) |
| `dict/industry.json` | 国民经济行业分类,三级联动 |
| `form/form.json` | 表单全局布局配置 |
| `form/field.json` | 各模块字段定义(INPUT/SELECT/DIVIDER 等,rules/editable/readable/mobile) |
| `region/region.json` | 中国行政区划树(国家→省→市→区县) |
| `task/message_task.json` | 消息任务模板,按模块定义事件+emailEnable/sysEnable/weComEnable |
| `i18n/cordys-crm_{zh_CN,en_US}.properties` | 后端国际化资源 |
| `migration/` | Flyway 数据库迁移脚本 |

### 7.2 数据库迁移(Flyway)

```
migration/
├── 1.0.0/  ddl/  V1.0.0_{1..6}__{init,system_setting,qrtz_schema,customer,clue,opportunity}.sql
│          dml/  V1.0.0_2_1__data.sql  V1.0.0_2_2__permission.sql
├── 1.0.1/  ...  1.9.0/  (30+ 版本)
└── 1.9.0/  ddl/  V1.9.0_1__init.sql  V1.9.0_2__ga_ddl.sql
           dml/  V1.9.0_2_1__data.sql  V1.9.0_2_2__permission.sql
```

命名规范:`V{版本号}_{多段序号}__{描述}.sql`

- **前缀** `V`(Versioned 迁移)
- **版本号**:主.次.修,与目录名一致
- **序号**:ddl `_1/_2`;dml `_2_1/_2_2/_2_3/_2_4`(data / permission / modify_tel / modify_log)
- **双下划线** `__` 分隔版本与描述

Flyway 配置(commons.properties):
```
spring.flyway.enabled=true  locations=classpath:migration  table=cordys_crm_version  baseline-version=0
```

---

## 8. 依赖关系

### 8.1 Maven 模块依赖

```
cordys-crm(root, pom, Spring Boot 3.5.14 parent)
├── frontend(由 frontend-maven-plugin 构建)
└── backend
    ├── framework(jar)
    ├── crm(jar)        → depends on framework
    └── app(Spring Boot Fat JAR) → depends on framework + crm
```

`app` 通过 `spring-boot-maven-plugin` + `loaderImplementation=CLASSIC` 打成 Fat JAR,启动类 `cn.cordys.Application`。`maven-antrun-plugin` 在 `generate-resources` 阶段将 `frontend/packages/{web,mobile}/dist` 复制到 `src/main/resources/static[ /mobile]`。

### 8.2 后端核心依赖

| 分类 | 依赖 |
|---|---|
| Spring | spring-boot-starter-aop / starter-jetty / starter-validation |
| Web / Swagger | springdoc-openapi-starter-webmvc-ui 2.8.16 |
| 邮件 | org.eclipse.angus:jakarta.mail 2.0.5 |
| 持久化 | mybatis-spring-boot-starter 3.0.5 + mysql-connector-j + flyway-core/flyway-mysql + pagehelper 6.1.1 |
| 安全 | shiro-spring-boot-web-starter / shiro-core / shiro-web / shiro-spring 2.1.0(jakarta classifier) |
| 缓存/Session | spring-session-data-redis + redisson-spring-boot-starter 3.52.0 |
| 通用 | commons-lang3 3.20 / commons-codec 1.20 / commons-collections4 4.5 / commons-text 1.15 / commons-io 2.21 / commons-compress 1.28 |
| 调度 | cn.cordys:quartz-spring-boot-starter 1.0.1 |
| Excel | cn.idev.excel:fastexcel 1.3.0 |
| JWT | com.auth0:java-jwt 3.12.1 |
| 构建辅助 | flatten-maven-plugin 1.6.0 / jacoco 0.8.12 / Lombok |
| 测试 | spring-boot-starter-test + testcontainers 2.0.3 + embedded-mysql/embedded-redis 3.1.13 |

### 8.3 前端核心依赖

| 分类 | 依赖 |
|---|---|
| 框架 | vue 3.5.22 / vue-router 4.5 / pinia 2.3 + persistedstate |
| 国际化 | vue-i18n 9.13 |
| HTTP | axios 1.7 |
| 图表 | echarts 6.1 + vue-echarts 6.7.3 |
| AI 聊天 | @ai-sdk/vue + ai + markdown-it + katex + highlight.js + dompurify |
| 工具 | @vueuse/core 10.11 / lodash-es 4.17 / dayjs 1.11 / mitt 3.0 / query-string 8.2 / element-china-area-data 6.1 |
| 加密/文件 | jsencrypt 3.3 / jspdf 4.2 / html2canvas-pro 1.5 / canvg 4.0 / localforage 1.10 |
| 兼容 | @vitejs/plugin-legacy 6.0 |
| DevDeps | ESLint 9 / Prettier / Stylelint / Husky 8 / commitlint 17 / lint-staged 13 / TS 5.9.3 / vue-tsc 3.1.4 / Tailwind 3.4 |

### 8.4 外部服务依赖

| 服务 | 用途 | 开关/配置 |
|---|---|---|
| MySQL | 业务 + Quartz | `mysql.embedded.enabled` / `spring.datasource.*` |
| Redis | Session + 缓存 + 发布订阅 | `redis.embedded.enabled` / `spring.data.redis.*` |
| MCP Server (8082) | AI 智能创建/录入/查重 | `mcp.embedded.enabled` / `cordys.crm.url` |
| Cockpit (8088) | 智能服务后端 | `cockpit.embedded.enabled` / `cockpit.base.url` |
| MaxKB | AI 知识库问答 | integration/agent 配置 |
| DataEase | BI 可视化 | integration/dataease 配置 |
| 钉钉/飞书/企微 | 组织同步 + 通知 + OAuth | integration/{dingtalk,lark,wecom} 配置 |
| 企查查 | 企业信息查询 | integration/qcc 配置 |
| SSO | 单点登录 | integration/sso 配置 |

### 8.5 仓库

- Maven 中央仓库 + `https://repository.fit2cloud.com/repository/cordys/`(Cordys 私服)

---

## 9. 项目运行方式

### 9.1 Docker 一键部署

```bash
docker run -d --name cordys-crm --restart unless-stopped \
  -p 8081:8081 -p 8082:8082 -v ~/cordys:/opt/cordys \
  1panel/cordys-crm
```

- **端口**:8081(Web/API)、8082(MCP)
- **默认账号**:`admin` / `CordysCRM`
- **访问**:`http://<服务器IP>:8081/`

### 9.2 容器启动流程(installer/shells/start-all.sh)

1. `init-directories.sh`:创建 `/opt/cordys/{data/{mysql,files,redis},conf/{mysql,redis},logs/{cordys-crm,mcp-server}}`,复制默认配置到 `conf/`(已存在则跳过),chmod 777
2. MySQL(`mysql.embedded.enabled=true` 时):`start-mysql.sh` + wait-for-it 3306
3. Redis:`start-redis.sh` + wait-for-it 6379
4. Cordys CRM:`start-cordys.sh` → `run-java.sh`(Java 21,JAVA_CLASSPATH=/app:/app/lib/*,JAVA_MAIN_CLASS=cn.cordys.Application)
5. MCP(后台):wait-for-it 8081 → `start-mcp.sh`(java -jar cordys-crm-mcp-server.jar),wait-for-it 8082
6. Cockpit(后台):wait-for-it 8082 → `start-cockpit.sh`,wait-for-it 8088

### 9.3 关键配置文件

**installer/conf/cordys-crm.properties**(All-in-One 默认,可覆盖 commons.properties):

```
mysql.embedded.enabled=true  redis.embedded.enabled=true  mcp.embedded.enabled=true
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/cordys-crm?...
spring.datasource.username=root  password=CordysCRM@mysql
spring.data.redis.host=127.0.0.1  port=6379  password=CordysCRM@redis
cordys.crm.url=http://127.0.0.1:8081
spring.ai.mcp.server.type=ASYNC  name=cordys-crm-mcp-server  version=1.0.0
allowed.ip.ranges.enabled=false
xss.protection.url.list=/account/follow/**,/announcement/add
cordys.secret.key=9a9rdqPlTqhpZzkq
```

**backend/app/src/main/resources/commons.properties**(应用默认):

```
server.port=8081  spring.application.name=cordys-crm
logging.file.path=/opt/cordys/logs/cordys-crm
spring.datasource.hikari.maximum-pool-size=100  minimum-idle=10
quartz.enabled=true  scheduler-name=cordys-crm-quartz  thread-count=10
mybatis.configuration.map-underscore-to-camel-case=true
spring.threads.virtual.enabled=true
spring.flyway.enabled=true  locations=classpath:migration  table=cordys_crm_version
spring.servlet.multipart.max-file-size=1024MB  max-request-size=1024MB
spring.session.timeout=43200s  spring.cache.type=redis
spring.messages.basename=i18n/cordys-crm
cockpit.base.url=http://cordys-crm-cockpit:8088  call-timeout=PT2M
```

### 9.4 源码本地构建

环境:JDK 21 + Node v22.16+ + pnpm 10.4.1

```bash
# 1. 父 POM 安装(多模块必要)
./mvnw install -N

# 2. 后端模块(framework/crm/app)
./mvnw clean install -DskipTests -DskipAntRunForJenkins --file backend/pom.xml

# 3. 前端(frontend/)
pnpm i -w          # 安装
npm run build      # 统一构建 web + mobile
# 或单独:
cd packages/web && npm run dev/build
cd packages/mobile && npm run dev/build

# 4. 整体打包(含前端 dist 复制)
./mvnw clean package
```

### 9.5 Dockerfile 多阶段构建

```
Stage 1(node:22-slim): frontend → pnpm build
Stage 2(eclipse-temurin:21-jdk): 复制前端 dist → mvnw clean package -DskipTests -pl '!frontend' → jar -xf 提取 Fat JAR 到 BOOT-INF/lib/classes + META-INF
Stage 3(ghcr.io/cordys-dev/cordys-base): 复制 class/lib/static → 复制 shells/conf/mcp/cockpit → ENV(JAVA_CLASSPATH/JAVA_MAIN_CLASS/JAVA_OPTIONS)→ VOLUME /opt/cordys → EXPOSE 8081/8082/3306 → ENTRYPOINT start-all.sh
```

### 9.6 mobile 调试技巧

1. 登录 `web`,复制 localStorage 中 `sessionId` 与 `csrfToken`
2. 粘贴到 `mobile` 页面 localStorage,刷新即可模拟登录态
3. 手机端:`我的` 页面短时间连续点击用户名区域 10 次,唤出 `Eruda` 调试工具

---

## 10. CI/CD 与工程规范

### 10.1 GitHub Workflows

| Workflow | 用途 |
|---|---|
| `build-and-push.yml` | 主构建 + 推送 Docker 镜像 |
| `build-and-push-base.yml` | 构建推送基础镜像 |
| `frontend-build.yml` | 前端独立构建 |
| `codecov.yml` | 代码覆盖率上报 |
| `add-labels-for-pr.yml` | PR 自动打标签 |
| `create-pr-from-push.yml` | push 自动建 PR |
| `issue-{open,comment,close,sync-to-tapd}.yml` | Issue 自动化 + TAPD 同步 |
| `llm-code-review.yml` | LLM 代码审查 |
| `sync2gitee.yml` | 同步 Gitee |
| `typos-check.yml` | 拼写检查 |

### 10.2 代码规范

- 后端:Lombok(lombok.config)+ JaCoCo(排除 mapper/domain/common/config/aspectj/mybatis/security)+ SonarCloud
- 前端:ESLint 9 + Prettier + Stylelint + Husky(commit-msg / pre-commit)+ commitlint(conventional)+ lint-staged + typos 检查
- 协议:FIT2CLOUD Open Source License(GPLv3 基础,附加 Logo/版权不可修改、衍生作品同样开源义务)

---

## 11. 关键设计总结

1. **分层架构**:`framework`(通用)→ `crm/common`(业务公共)→ `crm/crm/{模块}`(controller/service/mapper/domain 垂直分层)
2. **统一响应**:`ResultHolder` + `ResultResponseBodyAdvice` 自动包装 + `RestControllerExceptionHandler`
3. **安全体系**:Shiro(LocalRealm + 4 过滤器链)+ `@CsPermission` 切面 + `DataScopeService`
4. **字段加密**:`MybatisInterceptor` + `MybatisInterceptorConfig` 拦截 update/query,敏感字段 AES 加解密
5. **通用 Mapper**:`BaseMapper<E>` + 内置 SqlProvider + `LambdaQueryWrapper`,类型安全 Lambda 查询
6. **操作日志**:`@OperationLog` 注解 + `OperationLogAopAdvisor` 切面 + SpEL 解析 + `OperationLogHandler` 异步落库
7. **ID 生成**:Snowflake 改进版 `DefaultUidGenerator`(WorkerId 由 `DisposableWorkerIdAssigner` 数据库分配)
8. **文件存储**:`FileRepository` 接口 + `LocalRepository`/`S3Repository` 双实现 + `FileCenter` 工厂
9. **多租户**:`OrganizationContext` 贯穿
10. **第三方集成**:统一 `integration/`(MaxKB / DataEase / 钉钉 / 飞书 / 企微 / 企查查 / SSO / 招投标)
11. **前端 Monorepo**:`lib-shared` 共享 + `web`(Naive UI 全量)/ `mobile`(Vant UI 精简)双端
12. **i18n 就近定义**:`import.meta.glob` 自动聚合组件/页面 locale,无需集中维护
13. **AI 赋能**:MCP Server(8082)暴露 CRM Skills + 前端 `ai-chat` 组件(@ai-sdk/vue)
14. **All-in-One 部署**:内置 MySQL / Redis / MCP / Cockpit,`*.embedded.enabled` 灵活切换
15. **虚拟线程**:Java 21 `spring.threads.virtual.enabled=true`,IO 密集型吞吐提升

---

## 附录:关键文件路径速查

### 后端

- 启动类: [backend/app/src/main/java/cn/cordys/Application.java](file:///E:/工作/金数湾/CordysCRM/backend/app/src/main/java/cn/cordys/Application.java)
- 应用配置: [backend/app/src/main/resources/commons.properties](file:///E:/工作/金数湾/CordysCRM/backend/app/src/main/resources/commons.properties)
- 权限定义: [backend/crm/src/main/resources/permission.json](file:///E:/工作/金数湾/CordysCRM/backend/crm/src/main/resources/permission.json)
- 数据库迁移: [backend/crm/src/main/resources/migration](file:///E:/工作/金数湾/CordysCRM/backend/crm/src/main/resources/migration)
- 顶层 POM: [pom.xml](file:///E:/工作/金数湾/CordysCRM/pom.xml)

### 前端

- Monorepo 根: [frontend/package.json](file:///E:/工作/金数湾/CordysCRM/frontend/package.json)
- Axios 封装: [frontend/packages/lib-shared/api/http/Axios.ts](file:///E:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/http/Axios.ts)
- i18n 核心: [frontend/packages/lib-shared/locale/index.ts](file:///E:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/locale/index.ts)
- AI 聊天 Provider: [frontend/packages/lib-shared/ai-chat/AiChatProvider.vue](file:///E:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/ai-chat/AiChatProvider.vue)
- Web 入口: [frontend/packages/web/src/main.ts](file:///E:/工作/金数湾/CordysCRM/frontend/packages/web/src/main.ts)
- Web 路由: [frontend/packages/web/src/router/index.ts](file:///E:/工作/金数湾/CordysCRM/frontend/packages/web/src/router/index.ts)
- Mobile 入口: [frontend/packages/mobile/src/main.ts](file:///E:/工作/金数湾/CordysCRM/frontend/packages/mobile/src/main.ts)
- 前端 README: [frontend/REDEME.md](file:///E:/工作/金数湾/CordysCRM/frontend/REDEME.md)

### 部署

- Dockerfile: [installer/Dockerfile](file:///E:/工作/金数湾/CordysCRM/installer/Dockerfile)
- 启动脚本: [installer/shells/start-all.sh](file:///E:/工作/金数湾/CordysCRM/installer/shells/start-all.sh)
- 默认配置: [installer/conf/cordys-crm.properties](file:///E:/工作/金数湾/CordysCRM/installer/conf/cordys-crm.properties)
- 构建说明: [BUILD.md](file:///E:/工作/金数湾/CordysCRM/BUILD.md)
- 项目 README: [README.md](file:///E:/工作/金数湾/CordysCRM/README.md)

---

> 本 Wiki 基于仓库当前代码静态分析生成,实际行为以源码与最新 release 为准。最新进展请关注 [Cordys CRM 官网](https://cordys.cn/docs/) 与 [GitHub Releases](https://github.com/1Panel-dev/CordysCRM/releases)。
