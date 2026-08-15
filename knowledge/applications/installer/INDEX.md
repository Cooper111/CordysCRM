# 部署/安装器 · 索引（INDEX）

> 进入 installer/ 后的导航。

---

## 一、文件速查

| 文件 | 作用 | 何时读 |
|---|---|---|
| [application-installer.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/installer/application-installer.md) | 应用总览（本目录下已读） | 首次进入 installer 必读 |
| [shells/start-all.sh](file:///e:/工作/金数湾/CordysCRM/installer/shells/start-all.sh) | 总启动入口 | 启动顺序/依赖关系问题 |
| [shells/init-directories.sh](file:///e:/工作/金数湾/CordysCRM/installer/shells/init-directories.sh) | 目录初始化+配置复制 | 数据目录/权限问题 |
| [shells/start-cordys.sh](file:///e:/工作/金数湾/CordysCRM/installer/shells/start-cordys.sh) | CRM 启动 | JVM 参数/类路径问题 |
| [conf/cordys-crm.properties](file:///e:/工作/金数湾/CordysCRM/installer/conf/cordys-crm.properties) | 主配置文件模板 | 任何配置修改 |
| [Dockerfile](file:///e:/工作/金数湾/CordysCRM/installer/Dockerfile) | 主镜像 Dockerfile | 构建问题/添加依赖 |
| [Dockerfile.base](file:///e:/工作/金数湾/CordysCRM/installer/Dockerfile.base) | 基础镜像 | 基础 OS/JDK/工具链问题 |
| [mcp/mcp.md](file:///e:/工作/金数湾/CordysCRM/installer/mcp/mcp.md) | MCP 说明 | MCP Server 相关 |
| [cockpit/cockpit.md](file:///e:/工作/金数湾/CordysCRM/installer/cockpit/cockpit.md) | Cockpit 说明 | Cockpit 相关 |

---

## 二、常见问题路由

| 问题 | 检查位置 |
|---|---|
| 启动后 8081 无法访问 | start-cordys.sh 退出码 + logs/cordys-crm/*.log |
| MySQL 连接失败 | mysql.embedded.enabled + spring.datasource.* + 3306 端口监听 |
| Redis 连接失败 | redis.embedded.enabled + spring.data.redis.* + 6379 端口 |
| MCP 调用失败 | MCP 是否启动 + cordys.crm.url 是否能从 MCP 容器回连到 8081 |
| 文件上传后找不到 | 检查 /opt/cordys/data/files/ 挂载 + StorageType 配置 |
| 自定义系统密钥 | conf/cordys-crm.properties 中 cordys.secret.key（生产环境必改） |
| 内嵌→外置 MySQL/Redis | 把 2 个 embedded.enabled 置 false，填外部连接信息后重启容器 |
