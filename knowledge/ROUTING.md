# Cordys CRM · 知识路由（ROUTING）

> 收到需求后，AI **必须首先读取本文件**。通过提取关键词 → 匹配路由 → 定位到正确的知识入口，禁止直接全量扫描知识库。

---

## 一、路由执行步骤

```
步骤1: 从需求中提取关键词（见下方关键词分类）
步骤2: 匹配路由场景（Topic/状态/业务身份/模块/接口）
步骤3: 定位 → 候选业务域 → 候选应用 → 对应 INDEX.md
步骤4: 进入应用后按 INDEX 导航，按需读取 knowledge 文件
步骤5: 始终记住：回到代码核对易变项（见 KNOWLEDGE-RULES.md R4）
```

---

## 二、关键词提取清单

从需求描述、PRD、Bug 报告中主动提取以下信息：

| 提取维度 | 示例关键词 | 路由作用 |
|---|---|---|
| **业务模块** | 线索、客户、联系人、商机、合同、订单、产品、发票、收款、公海、线索池、跟进、仪表盘、审批、自定义表单、系统管理、智能体 | 定位 main/ 子模块 → applications 对应应用 |
| **业务身份/角色** | 销售、销售主管、管理员、普通用户、负责人、协作人、公海管理员 | 进入 main/business-rules.md → applications 的 solution 目录 |
| **状态/状态码** | 新建、已分配、跟进中、已转化、已赢单、已输单、已签约、已发货、已完成、已回收 | 进入 main/state-defs.md → 对应应用的 state-*.md |
| **接口/Topic** | API 路径、Controller 方法名、MQ Topic、消息事件 | 进入 applications/{app}/domain/base/api.md 或 msg.md |
| **技术关键词** | 登录、权限、加密、缓存、Session、Redis、MySQL、定时任务、导出、Excel、文件上传 | 进入 main/tech-constraints.md → applications/{app}/tech/ |
| **端标识** | PC端、Web端、移动端、手机端、H5、App | 定位 frontend-web / frontend-mobile / lib-shared |
| **运维/部署** | Docker、部署、启动、配置、端口、升级、安装、备份 | 定位 applications/installer |

---

## 三、典型路由场景

### 场景 A: 按业务模块路由（最常用）

> 下表关键词细化到 CRM 每个子模块的具体能力：出现对应关键词 → 直接定位到 applications/backend/domain/product/{模块}.md

| 需求中出现模块关键词（细化到能力点） | → 先读 main/ | → 再读应用 | → 必读 product 文档 | → 代码目录 |
|---|---|---|---|---|
| **线索** / 线索新建 / 线索编辑 / 线索删除 / 线索查询 / 线索列表 / 线索详情 / 线索导入 / 线索导出 / 线索批量编辑<br>**线索池** / 线索分配 / 线索领取 / 线索投放 / 公海线索 / 线索回收<br>**线索容量** / 容量上限 / 容量超限 / 容量配额<br>**线索转客户** / 线索转化 / 线索转客户失败 / 重复线索<br>**线索负责人** / 负责人历史 / 负责人变更 / 移交线索<br>**线索跟进** / 线索跟进计划 / 线索跟进记录 / 线索视图筛选 | [main/glossary.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/glossary.md) 线索术语 + [main/core-process.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/core-process.md) 线索生命周期 + [main/business-rules.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/business-rules.md) 容量+公海规则 + [main/state-defs.md](file:///e:/工作/金数湾/CordysCRM/knowledge/main/state-defs.md) 线索状态 | [backend/INDEX](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/INDEX.md) + [frontend-web/INDEX](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/frontend-web/INDEX.md) | [product/clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/clue.md) | `backend/crm/src/main/java/cn/cordys/crm/clue/` + `frontend/packages/web/src/views/clueManagement/` |
| **客户** / 客户新建 / 客户编辑 / 客户详情 / 客户列表 / 客户导入 / 客户导出 / 客户移交 / 客户批量编辑<br>**线索转客户** / 客户还原（反转化） / 客户合并 / 客户去重<br>**客户公海** / 客户回收 / 客户容量 / 客户分配 / 客户领取<br>**客户联系人** / 联系人新增 / 联系人修改 / 联系人去重 / 联系人查重<br>**客户协作** / 协作人 / 协作权限 / 只读协作 / 可编辑协作<br>**客户关系** / 关联公司 / 客户关系图 / 关系类型<br>**客户跟进** / 客户跟进计划 / 客户跟进记录 / 客户视图筛选 | main/glossary + core-process 客户生命周期 + business-rules 权限/容量/公海 + state-defs 客户状态 | backend + frontend-web (+ mobile) | [product/customer.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/customer.md) | `crm/customer/` + `web/views/customerManagement/` |
| **商机** / 商机新建 / 商机编辑 / 商机列表 / 商机详情 / 商机导入 / 商机导出 / 商机移交<br>**商机报价** / 报价单 / 报价版本 / 报价明细 / 报价打印<br>**商机阶段** / 阶段推进 / 阶段回退 / 阶段配置 / 高级阶段配置<br>**商机赢单** / 商机输单 / 赢输原因 / 商机规则（自动回退/回收）<br>**商机跟进** / 商机跟进计划 / 商机跟进记录 / 商机视图筛选 | main/glossary 商机术语 + core-process 商机跟进链路 + state-defs 商机阶段 | backend + frontend-web (+ mobile) | [product/opportunity.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/opportunity.md) | `crm/opportunity/` + `web/views/businessManagement/opportunity/` |
| **合同** / 合同新建 / 合同编辑 / 合同列表 / 合同详情 / 合同导入 / 合同导出 / 合同变更 / 合同作废 / 合同撤销<br>**合同阶段** / 合同阶段推进 / 阶段配置 / 合同审批提交<br>**工商抬头** / 抬头新增 / 抬头编辑 / 抬头审批 / 抬头启用停用<br>**合同发票** / 开票申请 / 发票列表 / 发票明细<br>**收款计划** / 回款计划 / 分期 / 计划完成 / 计划逾期<br>**收款记录** / 回款登记 / 收款凭证 / 部分收款<br>**合同快照** / 合同历史版本 / 合同对比 | main/glossary 合同术语 + core-process 合同→收款链路 + business-rules 审批触发 + state-defs 合同状态/工商抬头审批状态 | backend + frontend-web | [product/contract.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/contract.md) | `crm/contract/` + `web/views/contractManagement/` |
| **订单** / 订单新建 / 订单编辑 / 订单列表 / 订单详情 / 订单导入 / 订单导出 / 订单视图<br>**订单阶段** / 订单阶段推进 / 阶段配置 / 阶段排序<br>**订单快照** / 订单历史版本 / 订单执行跟踪 | main/glossary 订单术语 + core-process 订单执行链路 + state-defs 订单阶段 | backend + frontend-web | [product/order.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/order.md) | `crm/order/` + `web/views/orderManagement/` |
| **产品** / 产品新建 / 产品编辑 / 产品列表 / 产品详情 / 产品导入 / 产品导出 / 产品字段配置<br>**产品价格** / 价格版本 / 价格表 / 价格明细 / 价格审批 / 价格生效时间<br>**产品视图** / 产品筛选视图 | main/glossary 产品术语 + business-rules 权限 | backend + frontend-web | [product/product.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/product.md) | `crm/product/` + `web/views/productManagement/` |
| **跟进计划** / 计划新建 / 计划编辑 / 计划提醒 / 计划完成 / 计划延期 / 计划状态<br>**跟进记录** / 记录新建 / 记录编辑 / 记录附件 / 记录字段<br>**跟进评论** / 评论新增 / 评论回复 / 评论删除 / 评论 @提醒<br>跟进关联线索 / 跟进关联客户 / 跟进关联商机<br>跟进首页统计（待跟进/已逾期） | main/glossary 跟进术语 + core-process 跟进流程 + business-rules 通知触发 + state-defs 跟进计划状态 | backend + web/mobile | [product/follow.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/follow.md) | `crm/follow/` + `web/views/common/followUp/` + `web/views/home/` |
| **审批** / 审批流定义 / 审批流版本 / 审批启用停用 / 审批流程设计<br>**审批实例** / 发起审批 / 提交审批 / 审批详情 / 审批记录<br>**审批任务** / 审批待办 / 审批已办 / 我发起的 / 我审批的<br>**审批动作** / 通过 / 驳回 / 加签（前加/后加）/ 减签 / 退回 / 撤回 / 撤销 / 转办 / 催办<br>**审批抄送** / 审批节点条件 / 审批人配置（角色/部门/岗位/上级/发起人自选）<br>**审批WebHook** / 审批回调 / 审批事件推送 | main/business-rules.md 第四章 审批触发规则 + state-defs 审批状态 | backend + frontend-web | [product/approval.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/approval.md) | `crm/approval/` + `web/views/management-center/approval/` |
| **自定义表单** / 表单设计器 / 表单字段 / 表单布局 / 表单分组<br>**表单数据** / 表单数据新增 / 表单数据列表 / 表单数据详情 / 表单数据导出 / 表单数据编辑<br>**表单角色** / 表单管理员 / 表单数据权限 / 表单角色用户 | main/glossary 表单术语 | backend + frontend-web | [product/form.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/form.md) | `crm/form/` + `web/views/customForm/` |
| **仪表盘** / 仪表盘列表 / 仪表盘详情 / 仪表盘收藏 / 仪表盘新建 / 仪表盘编辑 / 仪表盘删除<br>**仪表盘模块** / 模块配置 / 模块拖拽排序 / 统计图表（柱状/折线/饼/漏斗）<br>仪表盘首页 / 工作台 | main/glossary 仪表盘术语 | backend + frontend-web | [product/dashboard.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/dashboard.md) | `crm/dashboard/` + `web/views/dashboard/` |
| **首页统计** / 首页 / 工作台 / 今日线索 / 今日客户 / 本周线索 / 本月线索 / 时间周期<br>线索统计图 / 客户统计图 / 销售漏斗 | main/core-process 销售漏斗 | backend + frontend-web + mobile | [product/home.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/home.md) | `crm/home/` + `web/views/home/` |
| **系统管理** / 用户 / 用户新建 / 用户编辑 / 用户停用 / 重置密码 / 个人中心<br>**角色** / 角色新建 / 角色编辑 / 角色权限 / 角色用户 / 数据权限范围<br>**部门** / 部门树 / 部门新建 / 部门重命名 / 部门负责人 / 部门排序<br>**组织** / 组织配置 / 切换组织 / 多租户 / 组织参数<br>**模块** / 模块管理 / 模块排序 / 模块启用停用 / 模块图标<br>**模块字段** / 字段配置 / 自定义字段 / 字段类型 / 字段必填 / 字段唯一性 / 模块表单<br>**字典** / 数据字典 / 枚举配置 / 字典项<br>**公告** / 公告发布 / 公告列表<br>**参数** / 系统参数 / 参数配置<br>**操作日志** / 操作记录 / 审计日志 / 日志查询 / 日志详情<br>**导出任务中心** / 导出进度 / 下载导出 / 导出失败 / 异步导出<br>**导入** / 数据导入 / 导入进度 / 导入失败 / 导入记录<br>**通知** / 消息通知 / 通知模板 / 通知发送 / 邮件发送 / 站内信<br>**调度任务** / 定时任务 / Quartz / Job / 手动触发 / 调度日志<br>**用户视图** / 列表视图 / 筛选视图 / 视图保存 / 视图切换 / 视图公开<br>**版权 / 版本信息** | main/business-rules.md 权限模型 + main/tech-constraints.md 后端约束 | backend + frontend-web | [product/system.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/system.md) | `crm/system/` + `web/views/management-center/` |
| **集成 / 第三方**<br>**MaxKB / 智能体** / AI Agent / 知识库 / 智能体模块 / 智能体分类<br>**DataEase** / BI 报表 / 报表嵌入 / 报表同步 / DataEase用户同步 / DataEase角色同步 / DataEase部门同步<br>**钉钉** / 钉钉消息推送 / 钉钉部门同步 / 钉钉扫码登录<br>**飞书** / 飞书消息 / 飞书部门同步 / 飞书审批<br>**企微 / 企业微信** / 企微消息 / 企微部门同步 / 企微通讯录<br>**企查查** / 工商查询 / 企业查询 / 企业详情 / 招投标 / 标讯<br>**SSO / OAuth** / 单点登录 / 第三方登录 / OAuth2 / 扫码登录<br>**第三方配置** / 通用配置 / 配置开关<br>**二维码** / 二维码生成 | main/core-process 集成部分 + tech-constraints 外部集成约束 | backend（+ web配置页） | [product/integration.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/integration.md) | `crm/integration/{agent,dataease,dingtalk,lark,wecom,qcc,sso,tender,sync,common}/` + `web/views/management-center/thirdConfig/` |

### 场景 B: 按状态码/状态流转路由

需求中出现状态关键词时（如"线索状态"、"商机阶段"、"合同状态"）：

```
第1步: 读 knowledge/main/state-defs.md
       ↓ 查对应模块的状态枚举定义、流转条件
第2步: 读 applications/backend/INDEX.md → domain/base/state-{module}.md
       ↓ 查状态在代码中的枚举类名、字段名
第3步: 回到代码核对实际枚举值
       backend/crm/src/main/java/cn/cordys/crm/{module}/domain/*.java
       + backend/crm/src/main/resources/form/field.json（前端字段配置）
```

### 场景 C: 按权限/角色路由

需求涉及"谁能做什么"、"数据可见范围"、"部门隔离"时：

```
第1步: 读 main/business-rules.md 的"权限模型与数据权限"章节
       ↓ 了解 Shiro + @CsPermission + DataScope 的整体设计
第2步: 读 applications/backend/INDEX.md → tech/security.md
       ↓ 了解 LocalRealm / AuthFilter / CsPermissionAspect 的实现方式
第3步: 读 backend/crm/src/main/resources/permission.json
       ↓ 确认权限节点定义
第4步: 回到代码核对具体 Controller 的 @CsPermission 注解
```

### 场景 D: 按接口/API 路由

需求中出现具体 API 路径或 Controller 方法名时：

```
第1步: 按路径前缀定位模块
       /api/clue/* → crm/clue
       /api/customer/* → crm/customer
       /api/opportunity/* → crm/opportunity
       ...
第2步: 读 applications/backend/domain/base/api-{module}.md
       ↓ 获取 Controller 类名、Service 入口
第3步: 直接读取代码
       backend/crm/src/main/java/cn/cordys/crm/{module}/controller/*.java
第4步: 同步查前端 API 封装
       frontend/packages/lib-shared/api/modules/{module}.ts
       frontend/packages/lib-shared/api/requrls/{module}/index.ts
```

### 场景 E: 按前端组件/页面路由

需求涉及前端页面、组件样式、交互时：

```
第1步: 按关键词定位端
       "PC端" / "后台" → frontend-web
       "移动端" / "手机" → frontend-mobile
       "共享组件" / "API封装" → lib-shared
第2步: 读对应 applications/{frontend-*}/INDEX.md
       ↓ 按 views/ 或 components/ 目录定位
第3步: 读取页面源码
       frontend/packages/{web,mobile}/src/views/{module}/*.vue
       frontend/packages/web/src/components/business/*.vue
第4步: 查相关 config/（页面配置）与 hooks/（复用逻辑）
```

### 场景 F: 按部署/运维关键词路由

```
需求关键词: Docker / 部署 / 启动 / 端口 / 配置 / MySQL / Redis / 升级
         ↓
第1步: 读 applications/installer/INDEX.md
       ↓ 了解 All-in-One 架构、内嵌服务、端口映射
第2步: 读 installer/conf/cordys-crm.properties
       ↓ 配置项说明
第3步: 读 installer/shells/*.sh
       ↓ 启动流程
```

---

## 四、路由冲突处理

当一个需求同时匹配多个路由场景时（如"修改线索状态时需要加审批"），按以下优先级处理：

| 优先级 | 场景类型 | 原因 |
|---|---|---|
| 1 | 业务模块路由 | 先定位到模块，再看其他维度 |
| 2 | 全局业务规则/权限 | 全局约束必须在模块实现之前理解 |
| 3 | 状态/接口/技术细节 | 最后落到实现细节 |

操作：先读最高优先级的知识，再依次读取其他维度，综合分析。

---

## 五、路由失败兜底

如果关键词匹配不到明确路由：

```
第1步: 读 knowledge/main/glossary.md
       ↓ 确认需求中术语的业务含义
第2步: 读 knowledge/main/core-process.md
       ↓ 判断需求落在哪个业务链路环节
第3步: 读对应 applications/{app}/INDEX.md 的全量目录
       ↓ 人工扫描匹配
第4步: 仍不确定时，向用户澄清（RD 流程的 clarify 阶段）
       不要凭猜测进入实现
```
