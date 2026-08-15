---
type: template
template-for: flow
status: official
updated: 2026-08-11
---

# 知识写作模板：流程（flow）

> 某条具体链路（如"线索转客户链路"、"合同审批链路"、"赢单到创建合同链路"）的详细流程。

---

## 复制以下内容到新文件：

```markdown
---
type: flow
app: backend
module: {模块名}
flow-name: {具体链路名，例如：线索转客户 / 赢单审批 / Excel导入线索}
status: official
evidence: {链路经过的关键类/方法路径，不少于3个}
owner: @{负责人}
verify-required: true
verify-note: 接口参数、DTO字段、状态枚举必须核对当前代码
updated: YYYY-MM-DD
related:
  - knowledge/main/core-process.md#{章节锚点}
  - knowledge/applications/backend/domain/base/api-{module}.md
---

# 流程：{具体链路中文名称}

> 场景：{一句话描述什么情况下会走这条链路}
> 触发入口（2选1填写）：
> - 前端: `{组件/Hook名}` → API `{URL}`
> - 定时任务: `{Job类名}` 触发 / MQ Consumer: `{Topic名}`

---

## 一、链路全览

```
[入口]
   │
   ▼
{Step1 控制器层} ──▶ {权限校验 CsPermission + DataScope}
   │
   ▼
{Step2 Service 主逻辑}
   ├─ 2.1 参数校验 / 状态校验
   ├─ 2.2 业务计算 / 状态流转
   ├─ 2.3 协作模块调用（同步）
   ├─ 2.4 DB 事务写入
   ├─ 2.5 发布消息 / 通知（异步，事务后）
   └─ 2.6 写入操作日志（AOP 自动）
   │
   ▼
[返回]
```

---

## 二、步骤详解（逐步展开）

### Step 1 · 控制器层

| 项 | 内容 |
|---|---|
| Controller 类 | `cn.cordys.crm.{module}.controller.{Name}Controller` |
| 方法签名 | 定位用（实际参数以当前代码为准）：`public ResultHolder<XXX> methodName(@RequestBody DTO dto)` |
| 权限注解 | `@CsPermission("{权限节点}")` |
| 日志注解 | `@OperationLog(operator=..., type=..., module=...)` |
| 审批注解 | `@HitApproval`（如有） |

### Step 2.x · 业务逻辑展开

| 子步骤 | 核心判断/计算 | 异常情况处理 | 关键代码入口 |
|---|---|---|---|
| 2.1 校验 | | | `{Service}.validateBeforeXxx()` |
| 2.2 计算 | | | |
| 2.3 协作 | | | |
| 2.4 DB | 事务边界在 {Service 方法名，@Transactional 标注处} | 回滚条件：RuntimeException + 手动 throw GenericException | |
| 2.5 消息 | 发送 Topic：{xxx} / 通知渠道：站内信+企微 | 失败不影响主流程 | |

---

## 三、关键 DTO / Domain（仅字段语义，实际值回代码核对）

| 对象 | 关键字段（**仅列业务关键语义**，不列全量字段） | 代码入口 |
|---|---|---|
| 请求 DTO | {字段语义列表} | `cn.cordys.crm.{module}.dto.XxxReqDTO` |
| 主 Domain | {字段语义列表} | `cn.cordys.crm.{module}.domain.Xxx` |
| 响应 DTO | {字段语义列表} | `cn.cordys.crm.{module}.dto.XxxRespDTO` |

---

## 四、数据库变更（涉及写库时必填）

| 表 | 操作（INSERT/UPDATE/DELETE） | 事务顺序 | 关键条件字段 |
|---|---|---|---|
| {表1} | | 1 | id / organization_id |
| {表2} | | 2 | |

---

## 五、幂等 & 并发策略

| 维度 | 策略 | 实现位置 |
|---|---|---|
| 幂等 | {唯一键 / 去重状态判断 / 防重 Token} | |
| 并发 | {字段乐观锁 version / 数据库行锁 / 分布式锁} | |

---

## 六、错误码 & 异常场景

| 错误场景 | 抛出的异常 / 返回错误码 | 前端提示文案策略 |
|---|---|---|
| 校验失败：状态不正确 | | |
| 校验失败：容量不足 | | |
| 校验失败：无权限 | 100403 FORBIDDEN | |
| 并发冲突 | | |
| 下游服务超时 | | |
