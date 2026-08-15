---
type: application
status: official
evidence: installer/ 目录源码
owner: cordys-crm
verify-required: true
updated: 2026-08-11
---

# 部署 / 安装器（installer）应用总览

> 代码路径：`installer/`
> 负责：Docker 镜像构建、All-in-One 部署、内嵌服务启动、配置文件

---

## 一、部署架构 All-in-One

```
                Docker 容器（单镜像 1panel/cordys-crm）
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  内置服务（可通过 *.embedded.enabled=false 切换为外部）：         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ MySQL 8  │  │ Redis    │  │ MCP      │  │ Cockpit  │       │
│  │ :3306    │  │ :6379    │  │ :8082    │  │ :8088    │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
│       │             │             │               │            │
│       └─────────────┴──────┬──────┴───────────────┘            │
│                            ▼                                   │
│              Cordys CRM Spring Boot (Fat JAR)                  │
│              端口 8081 (Web/API)                               │
│              Java 21 + 虚拟线程 (spring.threads.virtual=true)  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
         端口映射:   8081→8081   8082→8082
         数据卷:     ~/cordys:/opt/cordys
```

**一键运行**：
```bash
docker run -d --name cordys-crm --restart unless-stopped \
  -p 8081:8081 -p 8082:8082 -v ~/cordys:/opt/cordys \
  1panel/cordys-crm
```
默认账号: `admin` / `CordysCRM`，访问: `http://host:8081/`

---

## 二、目录结构

```
installer/
├── Dockerfile                # 主镜像 Dockerfile（多阶段构建）
├── Dockerfile.base           # 基础镜像 Dockerfile（ghcr.io/cordys-dev/cordys-base）
├── conf/                     # 配置文件模板（首次启动复制到 /opt/cordys/conf）
│   ├── cordys-crm.properties # 主配置（覆盖 commons.properties）
│   ├── mysql/my.cnf          # MySQL 配置
│   └── redis/redis.conf      # Redis 配置
├── shells/                   # 所有启动脚本
│   ├── init-directories.sh   # 创建目录 + 复制配置（每次启动先跑）
│   ├── start-all.sh          # 总入口（顺序启动 MySQL→Redis→CRM→MCP→Cockpit）
│   ├── start-mysql.sh        # MySQL 启动（内嵌时）
│   ├── start-redis.sh        # Redis 启动（内嵌时）
│   ├── start-cordys.sh       # CRM 启动（run-java.sh 方式）
│   ├── start-mcp.sh          # MCP Server 启动
│   ├── start-cockpit.sh      # Cockpit 启动
│   └── wait-for-it.sh        # 端口等待工具
├── mcp/mcp.md                # MCP 说明
└── cockpit/cockpit.md        # Cockpit 说明
```

---

## 三、Dockerfile 多阶段构建

```
Stage 1: node:22-slim
         └─ frontend/ → pnpm install → pnpm build → web dist + mobile dist

Stage 2: eclipse-temurin:21-jdk
         ├─ 复制 Stage 1 的 dist 到 backend/app/src/main/resources/static[ /mobile]
         ├─ ./mvnw clean package -DskipTests （后端打包 Fat JAR）
         └─ jar -xf 提取 BOOT-INF/lib/classes + META-INF（用于 CLASSIC 加载）

Stage 3: ghcr.io/cordys-dev/cordys-base
         ├─ 复制 Stage 2 的 class/lib/static
         ├─ 复制 shells/ + conf/ + mcp/ + cockpit/
         ├─ ENV(JAVA_CLASSPATH / JAVA_MAIN_CLASS / JAVA_OPTIONS)
         ├─ VOLUME /opt/cordys
         ├─ EXPOSE 8081/8082/3306
         └─ ENTRYPOINT [start-all.sh]
```

---

## 四、启动顺序（start-all.sh）

```
1. init-directories.sh
   ├─ 创建 /opt/cordys/{data/{mysql,files,redis},conf/{mysql,redis},logs/...}
   ├─ 复制 conf/ 模板到对应目录（已存在则跳过）
   └─ chmod 777 数据目录

2. MySQL（仅 mysql.embedded.enabled=true）
   └─ start-mysql.sh → wait-for-it :3306

3. Redis（仅 redis.embedded.enabled=true）
   └─ start-redis.sh → wait-for-it :6379

4. Cordys CRM（主应用）
   └─ start-cordys.sh → run-java.sh
      JAVA_MAIN_CLASS=cn.cordys.Application
      JAVA_CLASSPATH=/app:/app/lib/*
      JAVA_OPTIONS（JVM 参数，可通过环境变量覆盖）

5. MCP Server（后台，需 CRM 先启动）
   └─ wait-for-it :8081 → start-mcp.sh → wait-for-it :8082

6. Cockpit（后台，需 MCP 先启动）
   └─ wait-for-it :8082 → start-cockpit.sh → wait-for-it :8088
```

---

## 五、关键配置文件

### installer/conf/cordys-crm.properties（All-in-One 默认）

| 配置项 | 默认值 | 说明 |
|---|---|---|
| `mysql.embedded.enabled` | true | 是否启用内嵌 MySQL，false 用 spring.datasource 指向外部 |
| `redis.embedded.enabled` | true | 是否启用内嵌 Redis |
| `mcp.embedded.enabled` | true | 是否启动内嵌 MCP Server |
| `spring.datasource.url` | `jdbc:mysql://127.0.0.1:3306/cordys-crm` | MySQL 连接 |
| `spring.datasource.username` | root | |
| `spring.datasource.password` | CordysCRM@mysql | |
| `spring.data.redis.host` | 127.0.0.1 | Redis 主机 |
| `spring.data.redis.password` | CordysCRM@redis | |
| `cordys.crm.url` | http://127.0.0.1:8081 | MCP 回连 CRM 的地址 |
| `cockpit.base.url` | http://cordys-crm-cockpit:8088 | CRM 调 Cockpit 的地址 |
| `cordys.secret.key` | 9a9rdqPlTqhpZzkq | 系统级密钥（首次部署建议修改） |
| `allowed.ip.ranges.enabled` | false | 是否开启 IP 白名单限制 |
| `xss.protection.url.list` | `/account/follow/**,/announcement/add` | XSS 额外保护的 URL |

⚠️ **部署必读**：生产环境建议关闭内嵌 MySQL/Redis，改用外部高可用服务（把上面 2 个 embedded 置 false，填外部服务连接信息）。

---

## 六、AI 进入 installer 后的推荐路径

```
1. 读本文件 application-installer.md
2. 改配置 → 改 conf/cordys-crm.properties（改模板，不要改容器内文件）
3. 改启动流程 → 改 shells/*.sh
4. 改镜像构建 → 改 Dockerfile / Dockerfile.base
5. 验证配置是否生效：构建测试镜像 → 启动 → curl 健康检查
```
