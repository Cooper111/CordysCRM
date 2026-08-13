---
type: tech
status: official
evidence: pom.xml + package.json + commons.properties + CODE_WIKI.md
owner: cordys-crm
verify-required: true
verify-note: 版本号以 pom.xml / package.json 实际版本为准
updated: 2026-08-11
---

# Cordys CRM · 全局技术约束

> 本文件定义跨应用的全局技术选型、版本约束、编码约定。任何 AI 编码前必须先读本文件，避免引入错误技术栈或不兼容版本。

---

## 一、后端技术栈（强制）

### 1.1 核心版本

| 分类 | 技术 / 版本 | 代码核对位置 |
|---|---|---|
| **语言 / 运行时** | **Java 21**（虚拟线程已开启） | `pom.xml` → `<java.version>` |
| **应用框架** | **Spring Boot 3.5.x**（基于 Jetty，**排除 Tomcat**） | `pom.xml` → Spring Boot BOM |
| **安全框架** | **Apache Shiro 2.1.x**（Jakarta classifier） | `backend/crm/pom.xml` |
| **ORM** | **MyBatis Spring Boot Starter 3.0.x** + **PageHelper 6.1.x** | `backend/framework/pom.xml` |
| **数据库** | **MySQL 8** + **Flyway**（迁移脚本管理） | Flyway 配置见 commons.properties |
| **缓存 / Session** | **Redis** + **Redisson 3.52.x** + **spring-session-data-redis** | `backend/crm/pom.xml` |
| **调度** | **Quartz**（`cn.cordys:quartz-spring-boot-starter 1.0.1`） | `backend/app/pom.xml` |
| **API 文档** | **springdoc-openapi-starter-webmvc-ui 2.8.x** | `backend/crm/pom.xml` |
| **Excel** | **FastExcel 1.3.x**（cn.idev.excel） | `backend/framework/pom.xml` |
| **JWT** | **com.auth0:java-jwt 3.12.1** | `backend/crm/pom.xml` |
| **构建** | **Maven** + **flatten-maven-plugin** + **JaCoCo** | 根 `pom.xml` |

### 1.2 后端编码铁律

1. **必须使用 BaseMapper + LambdaQueryWrapper**，禁止手写原生 SQL（除非复杂统计场景）
   ```java
   // ✅ 正确：类型安全 Lambda 查询
   userMapper.selectListByLambda(
       new LambdaQueryWrapper<User>()
           .eq(User::getId, userId)
           .eq(User::getStatus, "ACTIVE")
   );
   ```

2. **统一响应 ResultHolder**：Controller 返回值必须用 `ResultHolder.success(data)` 或抛出 `GenericException`
   - 不要手动 new ResponseEntity
   - 不要返回裸对象（`ResultResponseBodyAdvice` 会自动包装，但建议显式使用）

3. **异常处理**：业务异常用 `GenericException` + `IResultCode`，不要抛出 `RuntimeException`
   - 全局异常由 `RestControllerExceptionHandler` 处理
   - 自定义错误码继承 `CrmHttpResultCode`

4. **字段加解密**：敏感字段（手机号、身份证等）通过 `MybatisInterceptor` 自动 AES 加解密
   - 新增/修改时：不要手动加密
   - 查询时：不要手动解密
   - 敏感字段清单见 `MybatisInterceptorConfig`

5. **UID 生成**：所有业务主键使用 `DefaultUidGenerator.getUID()`（Snowflake 改进版）
   - ❌ 禁止用自增 ID 作为业务主键
   - ❌ 禁止用 UUID（无序，索引性能差）

6. **写操作必须加 @OperationLog**：见 business-rules.md 第六章

7. **权限注解**：写操作 Controller 方法必须加 `@CsPermission("权限节点key")`

8. **虚拟线程**：`spring.threads.virtual.enabled=true` 已开启
   - IO 密集型操作无需额外配置
   - CPU 密集型建议手动提交到独立线程池

9. **文件存储**：使用 `FileCenter.getRepository(storageType)` 获取存储引擎
   - 支持 LOCAL（本地）和 S3（MinIO/阿里 OSS 等）
   - ❌ 禁止直接 new FileOutputStream 写磁盘

10. **Session / 当前用户**：通过 `SessionUtils.getUserId()` / `SessionUtils.getUser()` 获取
    - ❌ 禁止从请求参数或前端传递用户 ID 作为当前用户使用

---

## 二、前端技术栈（强制）

### 2.1 核心版本

| 分类 | 技术 / 版本 | 代码核对位置 |
|---|---|---|
| **框架** | **Vue 3.5.x** + **TypeScript 5.9.x** | `frontend/packages/*/package.json` |
| **构建** | **Vite** + **@vitejs/plugin-legacy**（兼容非 IE11） | `frontend/packages/*/config/vite.config.*.ts` |
| **状态管理** | **Pinia 2.3** + **pinia-plugin-persistedstate**（持久化） | `frontend/packages/*/package.json` |
| **路由** | **Vue Router 4.5**（**hash 模式**） | `frontend/packages/*/src/router/index.ts` |
| **国际化** | **vue-i18n 9.13**（组合式 API，legacy:false） | `frontend/packages/lib-shared/locale/index.ts` |
| **PC 端 UI** | **Naive UI** | `frontend/packages/web/package.json` |
| **移动端 UI** | **Vant UI** + `postcss-pxtorem`（rem 适配） | `frontend/packages/mobile/package.json` |
| **HTTP** | **axios 1.7**（`@lib/shared/api/http` 封装） | `frontend/packages/lib-shared/api/http/Axios.ts` |
| **图表** | **echarts 6.1** + **vue-echarts** | `frontend/packages/web/package.json` |
| **AI 聊天** | **@ai-sdk/vue** + **ai**（markdown-it + katex + highlight.js + dompurify） | `frontend/packages/lib-shared/ai-chat/` |
| **包管理** | **pnpm 10.4.x**（**monorepo workspace**，禁止 npm/yarn） | `frontend/pnpm-workspace.yaml` |
| **代码规范** | **ESLint 9** + **Prettier** + **Stylelint** + **Husky** + **commitlint** | `frontend/package.json` |
| **样式** | **Tailwind CSS 3.4** + **Less** | 各包下配置文件 |

### 2.2 前端编码铁律

1. **Monorepo 包结构**：
   ```
   frontend/
   ├── packages/lib-shared/   # @lib/shared：API、枚举、TS类型、工具、i18n、ai-chat
   ├── packages/web/          # @cordys/web：PC 端（Naive UI）
   └── packages/mobile/       # @cordys/mobile：移动端（Vant UI）
   ```
   - ✅ API 请求封装统一放 `lib-shared/api/modules/`，两端共用
   - ✅ TS 类型定义统一放 `lib-shared/models/`
   - ✅ 枚举统一放 `lib-shared/enums/`
   - ❌ 禁止在 web/mobile 中重复定义 API、枚举、TS 类型

2. **HTTP 请求**：使用 `@lib/shared/api` 封装，不要直接 new axios
   ```typescript
   // ✅ 正确
   import { getClueList } from '@lib/shared/api/modules/clue';
   const { data } = await getClueList(params);
   ```

3. **国际化 i18n**：
   - 切换语言：`useLocale().changeLocale(locale)`
   - 文案翻译：`t('key.path')`，不要硬编码中文字符串
   - 组件/页面级语言包：就近写在组件目录下的 `locale/zh-CN.ts` + `locale/en-US.ts`
   - 自动聚合：`import.meta.glob('../../{components,views}/**/locale/*.ts')`

4. **路由模式**：hash 模式（`#/clue/list`），不要使用 history 模式（后端配合成本高）

5. **权限判断**：使用 `usePermission()` hook + `v-permission` 指令
   ```vue
   <!-- 模板中 -->
   <n-button v-permission="'CLUE:ADD'">新增线索</n-button>
   ```
   ```typescript
   // 代码中
   const { hasPermission } = usePermission();
   if (hasPermission('CLUE:UPDATE')) { /* ... */ }
   ```

6. **表单创建**：使用 `useFormCreateApi()` hook（基于自定义表单引擎 `crm-form-create` 组件）
   - 字段定义来自后端 `form/field.json` + `form/form.json`
   - 不要手写大量表单 DOM（维护成本高）

7. **Husky + commitlint**：
   - Git 提交信息格式必须符合 Conventional Commits
   - `feat:` / `fix:` / `docs:` / `style:` / `refactor:` / `test:` / `chore:`
   - ❌ 禁止 `--no-verify` 绕过 husky

8. **Pinia store**：
   - 用户态 / 应用态用 Pinia，不要滥用 localStorage
   - 需持久化的 state 加 `persist: true`（pinia-plugin-persistedstate）

---

## 三、数据库与数据迁移约束

### 3.1 Flyway 迁移脚本规范

- 脚本目录：`backend/crm/src/main/resources/migration/{版本号}/`
- 命名规范：`V{版本号}_{多段序号}__{描述}.sql`
  ```
  V1.9.0_1__init.sql          # DDL 第一段
  V1.9.0_2_1__data.sql        # DML 数据初始化
  V1.9.0_2_2__permission.sql  # DML 权限初始化
  ```
- **禁止修改已执行的迁移脚本**（Flyway checksum 校验失败会导致启动失败）
- **必须增量新增**脚本，不要回退修改
- DDL 脚本中禁止使用数据库专有语法（仅 MySQL 8 兼容语法）

### 3.2 表与字段约束

- 所有表必须含：`id`(bigint)、`create_by`、`create_time`、`update_by`、`update_time`、`organization_id`、`deleted`(逻辑删除)
- 业务主键 ID 用 Snowflake UID（bigint），禁止自增 ID
- 金额字段用 `DECIMAL(18,2)`，禁止 float / double
- 状态/枚举字段用 `VARCHAR(32)` 存枚举字符串（可读性强，方便扩展）
- 逻辑删除字段：`deleted` (0=未删除, 1=已删除)
- 所有查询默认拼接 `deleted = 0`（MyBatis BaseMapper 处理）

---

## 四、中间件与部署约束

### 4.1 端口约定

| 服务 | 端口 | 说明 |
|---|---|---|
| Cordys CRM (Web/API) | **8081** | Spring Boot 主应用 |
| MCP Server | **8082** | AI Skills 服务 |
| Cockpit | **8088** | 智能服务后端 |
| MySQL (内嵌) | **3306** | 可切外置 |
| Redis (内嵌) | **6379** | 可切外置 |

### 4.2 All-in-One 部署开关

`installer/conf/cordys-crm.properties` 中控制：
| 配置项 | 含义 | true=内嵌，false=外置 |
|---|---|---|
| `mysql.embedded.enabled` | 内嵌 MySQL 开关 | 默认 true |
| `redis.embedded.enabled` | 内嵌 Redis 开关 | 默认 true |
| `mcp.embedded.enabled` | 内嵌 MCP Server 开关 | 默认 true |
| `cockpit.embedded.enabled` | 内嵌 Cockpit 开关 | 一般 true |

### 4.3 关键配置文件路径（AI 编码时常见配置项）

| 配置文件 | 作用 |
|---|---|
| `backend/app/src/main/resources/commons.properties` | 应用默认配置（端口、连接池、Flyway、Session TTL 等） |
| `installer/conf/cordys-crm.properties` | All-in-One 部署配置（覆盖上面的默认值） |
| `backend/crm/src/main/resources/permission.json` | 权限定义树 |
| `backend/crm/src/main/resources/form/field.json` | 各模块字段定义 |
| `backend/crm/src/main/resources/form/form.json` | 表单布局配置 |
| `backend/crm/src/main/resources/task/message_task.json` | 消息任务（通知渠道）配置 |
| `backend/crm/src/main/resources/dict/industry.json` | 行业分类字典 |
| `backend/crm/src/main/resources/region/region.json` | 行政区划树 |

---

## 五、AI / MCP 相关约束

### 5.1 MCP Server 架构

```
AI 助手（WorkBuddy / OpenClaw 等）
    │  调用 MCP Skills
    ▼
MCP Server (8082)
    │  HTTP API 调用
    ▼
Cordys CRM (8081) /api/agent/** → AgentSkillService
    │
    ▼
业务 Service（线索创建/查重/录入...）
```

### 5.2 AI 聊天组件

- 前端入口：`lib-shared/ai-chat/AiChatProvider.vue`
- 依赖：`@ai-sdk/vue` + `ai` SDK
- Markdown 渲染：`markdown-it` + `katex`（公式）+ `highlight.js`（代码高亮）+ `dompurify`（XSS 防护）
- ⚠️ 所有 AI 返回的 HTML 必须走 dompurify，禁止直接 `v-html` 未净化内容

---

## 六、测试与质量约束

### 6.1 后端测试

- 框架：`spring-boot-starter-test` + **Testcontainers**（embedded-mysql / embedded-redis）
- 测试必须带真实 MySQL/Redis（Testcontainers 自动拉取镜像）
- 不要用 H2 内存库替代 MySQL（SQL 方言差异会掩盖问题）
- JaCoCo 覆盖率：构建时自动统计，排除 mapper/domain/common/config/aspectj/mybatis/security

### 6.2 前端规范检查

- `pnpm lint`：ESLint + Prettier + Stylelint 检查
- `pnpm typecheck`：vue-tsc 类型检查
- `pnpm typos`：拼写检查（typos-cli）
- Husky `pre-commit` 钩子已集成上述检查
