# 后端应用知识 · 索引（INDEX）

> AI 进入后端应用后首先读取本文件，按路由导航按需加载，禁止全量扫描。

---

## 一、强制读取顺序

```
application-backend.md（已读）
    ↓
domain/product/{module}.md（按模块读主干能力）
    ↓
domain/solution/{special-case}.md（如有差异化逻辑）
    ↓
domain/base/api-{module}.md / domain-{module}.md / repository-{module}.md
    ↓
tech/{topic}.md（规范/踩坑）
    ↓
回到代码仓库实际读取源文件
```

---

## 二、domain/product/ 产品能力（主干流程）

> 注意：每个产品文档的「代码入口」列均为**定位路径参考**，易变项（方法/字段/枚举值）必须回到代码核对（KNOWLEDGE-RULES R4）。

| 文件 | 对应模块 · 代码包前缀 | 何时读取 | 核心 Service / Controller 入口（定位用） |
|---|---|---|---|
| [domain/product/clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/clue.md) | 线索模块<br>`cn.cordys.crm.clue.*` | 涉及线索 CRUD、线索分配、领取、转客户、容量、线索池、导入导出、负责人历史、视图筛选、跟进计划/记录时 | 核心 Service：ClueService、PoolClueService、CluePoolService、ClueCapacityService、ClueExportService、ClueOwnerHistoryService<br>核心 Controller：ClueController、PoolClueController、CluePoolController、ClueCapacityController、ClueOwnerHistoryController、PoolClueUserViewController、ClueUserViewController、ClueFollowPlanController、ClueFollowRecordController |
| [domain/product/customer.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/customer.md) | 客户模块<br>`cn.cordys.crm.customer.*` | 涉及客户/联系人/协作/公海/移交/容量/线索转客户/客户合并/导入导出/联系人查重时 | 核心 Service：CustomerService、CustomerContactService、CustomerCollaborationService、CustomerCapacityService、CustomerPoolService、PoolCustomerService、CustomerRelationService、CustomerExportService<br>核心 Controller：CustomerController、CustomerContactController、CustomerCollaborationController、CustomerPoolController、CustomerFollowPlanController、CustomerFollowRecordController、CustomerUserViewController |
| [domain/product/opportunity.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/opportunity.md) | 商机模块<br>`cn.cordys.crm.opportunity.*` | 涉及商机/报价/阶段推进/赢单输单/商机规则/导入导出/阶段配置/跟进时 | 核心 Service：OpportunityService、OpportunityQuotationService、OpportunityStageService、OpportunityRuleService、OpportunityExportService<br>核心 Controller：OpportunityController、OpportunityQuotationController、OpportunityStageController、OpportunityRuleController、OpportunityUserViewController、OpportunityQuotationUserViewController、OpportunityFollowPlanController、OpportunityFollowRecordController |
| [domain/product/contract.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/contract.md) | 合同模块<br>`cn.cordys.crm.contract.*` | 涉及合同/工商抬头/合同发票/收款计划/收款记录/合同阶段/作废/导入导出/快照时 | 核心 Service：ContractService、BusinessTitleService、ContractInvoiceService、ContractStageService、ContractExportService<br>核心 Controller：ContractController、ContractInvoiceController、BusinessTitleController |
| [domain/product/order.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/order.md) | 订单模块<br>`cn.cordys.crm.order.*` | 涉及订单/订单阶段配置/订单导出/订单阶段推进/快照时 | 核心 Service：OrderService、OrderStageService、OrderExportService<br>核心 Controller：OrderController、OrderUserViewController |
| [domain/product/product.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/product.md) | 产品模块<br>`cn.cordys.crm.product.*` | 涉及产品/价格/产品字段配置/价格版本/导入导出时 | 核心 Service：ProductService、ProductPriceService、ProductExportService<br>核心 Controller：ProductController、ProductPriceController、ProductUserViewController |
| [domain/product/follow.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/follow.md) | 跟进模块<br>`cn.cordys.crm.follow.*` | 涉及跟进计划/跟进记录/评论/跟进提醒/计划状态流转/关联线索客户商机时 | 核心 Service：FollowUpPlanService、FollowUpRecordService、BaseFollowUpService、FollowUpPlanCommentService、FollowUpRecordCommentService、FollowUpPlanLogService、FollowUpRecordLogService<br>核心 Controller：FollowUpPlanController、FollowUpRecordController、FollowUpPlanCommentController、FollowUpRecordCommentController、FollowUpPlanUserViewController |
| [domain/product/approval.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/approval.md) | 审批流<br>`cn.cordys.crm.approval.*` | 涉及审批流定义/版本/实例/节点/任务/加签/退回/抄送/审批动作/审批资源处理器/WebHook 时 | 核心 Service：ApprovalFlowService、ApprovalInstanceService、ApprovalTaskService（待办）、ApprovalActionService（审批动作）、ApprovalTodoService、ApprovalResourceService、ApprovalFlowLogService<br>Controller：统一在业务模块入口触发（如合同提交审批），审批动作通过 ApprovalAction* 接口 |
| [domain/product/form.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/form.md) | 自定义表单<br>`cn.cordys.crm.form.*` | 涉及表单设计/表单数据/表单数据导出/表单角色/表单权限/字段Blob扩展时 | 核心 Service：CustomFormService、CustomFormDataService、CustomFormRoleService、CustomFormDataFieldService、CustomFormDataExportService<br>核心 Controller：CustomFormController、CustomFormDataController、CustomFormRoleController |
| [domain/product/dashboard.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/dashboard.md) | 仪表盘<br>`cn.cordys.crm.dashboard.*` | 涉及仪表盘/仪表盘收藏/仪表盘模块/模块排序/自定义看板时 | 核心 Service：DashboardService、DashboardModuleService、DashboardSortService<br>核心 Controller：DashboardController |
| [domain/product/home.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/home.md) | 首页统计<br>`cn.cordys.crm.home.*` | 涉及首页统计/线索统计/客户统计/时间周期筛选（今日/本周/本月）时 | 核心 Service：HomeStatisticService<br>核心 Controller：HomeStatisticController |
| [domain/product/system.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/system.md) | 系统管理<br>`cn.cordys.crm.system.*` | 涉及用户/角色/部门/组织/权限/模块/字典/公告/参数/操作日志/导出任务中心/导入任务/个人中心/用户视图/定时任务/调度表/模块字段/模块表单/组织配置/消息通知 时 | 核心 Service：UserService、RoleService、DepartmentService、RoleUserService、ModuleService、ModuleFieldService、ModuleFormService、DictService、ExportTaskService、NoticeService、ScheduleService、ParameterService、PersonalCenterService、UserViewService、OrganizationConfigService<br>核心 Controller：UserController、RoleController、DepartmentController、ModuleController、ModuleFieldController、PersonalCenterController 等（见 base/api-system.md） |
| [domain/product/integration.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/product/integration.md) | 第三方集成<br>`cn.cordys.crm.integration.*` | 涉及 MaxKB Agent、DataEase 同步、钉钉/飞书/企微部门同步与消息推送、企查查企业查询、SSO登录、招投标、第三方通用配置 时 | 子包：agent / dataease / dingtalk / lark / wecom / qcc / sso / tender / sync / common<br>核心 Service：AgentBaseService、AgentModuleService、DataEaseSyncService、DataEaseService、LarkDepartmentService、DingTalkDepartmentService、WeComDepartmentService、SSOService、OAuthStateService + 各类 NoticeSender |

---

## 三、domain/solution/ 解决方案差异（如有）

> 产品通用能力在上表 product 中。若有针对特定业务线的差异化扩展，写在 solution/ 中。当前版本通用实现，solution 暂为占位。

| 文件 | 说明 |
|---|---|
| `solution/default.md` | 默认方案（与 product 一致） |

---

## 四、domain/base/ 基础索引（高频定位）

> 本部分为"易变项定位入口"，只写类名/路径前缀，不写具体方法签名和字段（易变，回到代码核对，KNOWLEDGE-RULES R4 & W4）。

### 4.1 API 索引（Controller 入口）

| 文件 | 对应模块 | 代码定位路径前缀 / 核心 Controller 类名（定位用） |
|---|---|---|
| [domain/base/api-clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-clue.md) | 线索 API | `cn.cordys.crm.clue.controller` 下：ClueController、PoolClueController、CluePoolController、ClueCapacityController、ClueOwnerHistoryController、PoolClueUserViewController、ClueUserViewController、ClueFollowPlanController、ClueFollowRecordController |
| [domain/base/api-customer.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-customer.md) | 客户 API | `cn.cordys.crm.customer.controller` 下：CustomerController（含线索转客户/合并/移交）、CustomerContactController、CustomerCollaborationController、CustomerPoolController、CustomerFollowPlanController、CustomerFollowRecordController、CustomerUserViewController |
| [domain/base/api-opportunity.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-opportunity.md) | 商机 API | `cn.cordys.crm.opportunity.controller` 下：OpportunityController、OpportunityQuotationController、OpportunityStageController、OpportunityRuleController、OpportunityUserViewController、OpportunityQuotationUserViewController、OpportunityFollowPlanController、OpportunityFollowRecordController |
| [domain/base/api-contract.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-contract.md) | 合同 API | `cn.cordys.crm.contract.controller` 下：ContractController（含阶段推进/作废/工商抬头）、ContractInvoiceController（发票/开票/收款计划/收款记录）、BusinessTitleController |
| [domain/base/api-order.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-order.md) | 订单 API | `cn.cordys.crm.order.controller` 下：OrderController（含阶段推进/导入导出/快照）、OrderUserViewController |
| [domain/base/api-product.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-product.md) | 产品 API | `cn.cordys.crm.product.controller` 下：ProductController、ProductPriceController、ProductUserViewController |
| [domain/base/api-follow.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-follow.md) | 跟进 API | `cn.cordys.crm.follow.controller` 下：FollowUpPlanController、FollowUpRecordController、FollowUpPlanCommentController、FollowUpRecordCommentController、FollowUpPlanUserViewController |
| [domain/base/api-approval.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-approval.md) | 审批 API | 审批动作接口（提交/通过/驳回/加签/撤回/撤销）位于各业务 Controller；审批流配置/待办/已办/抄送等独立审批接口：ApprovalFlowController、ApprovalInstanceController、ApprovalTodoController、ApprovalActionController |
| [domain/base/api-form.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-form.md) | 自定义表单 API | `cn.cordys.crm.form.controller` 下：CustomFormController、CustomFormDataController、CustomFormRoleController |
| [domain/base/api-dashboard.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-dashboard.md) | 仪表盘 API | `cn.cordys.crm.dashboard.controller.DashboardController` |
| [domain/base/api-home.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-home.md) | 首页统计 API | `cn.cordys.crm.home.controller.HomeStatisticController`（今日/本周/本月 线索/客户统计） |
| [domain/base/api-system.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-system.md) | 系统 API（最大） | `cn.cordys.crm.system.controller` 下：User、Role、Department、Module、ModuleField、ModuleForm、Dict、ExportTask、Import、Notice、Announcement、Schedule、Parameter、PersonalCenter、UserView、OrganizationConfig、OperationLog 等 20+ 个 Controller |
| [domain/base/api-integration.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/api-integration.md) | 集成 API | AgentController / AgentModuleController / TenderController / SSOController，其他集成以 Service 内部调用为主（DataEase/钉钉/飞书/企微/企查查） |

### 4.2 Domain 领域模型索引

> 仅定位类名，字段/注解以代码为准。所有业务 Domain 继承 BaseAutoIdEntity 或其子类（见 repository-core.md）。

| 文件 | 内容 | 代码定位 · 核心 Domain 类名（定位用） |
|---|---|---|
| [domain/base/domain-clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-clue.md) | 线索/线索池/线索容量/线索字段/线索负责人 领域类 | `cn.cordys.crm.clue.domain` 下：Clue、CluePool、CluePoolPickRule、CluePoolRecycleRule、CluePoolHiddenField、ClueCapacity、ClueOwner、ClueField、ClueFieldBlob |
| [domain/base/domain-customer.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-customer.md) | 客户/联系人/公海/协作/关系/容量 领域类 | `cn.cordys.crm.customer.domain` 下：Customer、CustomerContact、CustomerContactField、CustomerContactFieldBlob、CustomerField、CustomerFieldBlob、CustomerPool、CustomerPoolPickRule、CustomerPoolRecycleRule、CustomerPoolHiddenField、CustomerCapacity、CustomerOwner、CustomerCollaboration、CustomerRelation |
| [domain/base/domain-opportunity.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-opportunity.md) | 商机/报价/阶段配置/阶段高级配置 领域类 | `cn.cordys.crm.opportunity.domain` 下：Opportunity、OpportunityQuotation、OpportunityStageConfig、OpportunityRule + 对应 Field/FieldBlob/Snapshot 类 |
| [domain/base/domain-contract.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-contract.md) | 合同/发票/收款计划/收款记录/工商抬头/快照 领域类 | `cn.cordys.crm.contract.domain` 下：Contract、ContractField、ContractFieldBlob、ContractSnapshot、ContractInvoice、ContractInvoiceField、ContractInvoiceFieldBlob、ContractInvoiceSnapshot、ContractPaymentPlan、ContractPaymentPlanField、ContractPaymentRecord、BusinessTitle、BusinessTitleConfig、ContractStageConfig |
| [domain/base/domain-order.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-order.md) | 订单/阶段配置/快照 领域类 | `cn.cordys.crm.order.domain` 下：Order、OrderField、OrderFieldBlob、OrderSnapshot、OrderStageConfig |
| [domain/base/domain-product.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-product.md) | 产品/价格/字段 领域类 | `cn.cordys.crm.product.domain` 下：Product、ProductField、ProductFieldBlob、Price、PriceField、PriceFieldBlob |
| [domain/base/domain-follow.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-follow.md) | 跟进计划/跟进记录/评论/字段 领域类 | `cn.cordys.crm.follow.domain` 下：FollowUpPlan、FollowUpPlanField、FollowUpPlanFieldBlob、FollowUpRecord、FollowUpRecordField、FollowUpRecordFieldBlob、FollowUpPlanComment、FollowUpRecordComment、Comment |
| [domain/base/domain-approval.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-approval.md) | 审批流/版本/实例/节点/链接/任务/加签/记录/快照/退回记录 领域类 | `cn.cordys.crm.approval.domain` 下：ApprovalFlow、ApprovalFlowVersion、ApprovalInstance、ApprovalNode、ApprovalNodeApprover、ApprovalNodeCondition、ApprovalNodeLink、ApprovalTask、ApprovalAddSignTask、ApprovalRecord、ApprovalResourceSnapshot、ApprovalReturnBackRecord |
| [domain/base/domain-system.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/domain-system.md) | 用户/角色/部门/组织/权限/模块/字段/字典/公告/参数/操作日志/导出/调度/视图/组织配置 领域类 | `cn.cordys.crm.system.domain` 下：User、Role、Department、UserRole、RolePermission、RoleScopeDept、Organization、OrganizationUser、OrganizationConfig、OrganizationConfigDetail、Module、ModuleField、ModuleFieldBlob、OperationLog、OperationLogBlob、Dict、Announcement、Parameter、ExportTask、Schedule、UserView、UserViewCondition、WorkerNode 等 30+ 类 |

### 4.3 Repository / Mapper / 表索引

> 只写 Mapper 类定位入口 + 对应表名的命名规律，具体 SQL 和字段看 Mapper.xml。
> Mapper 规范：一律使用 `Ext{Entity}Mapper` 继承 `BaseMapper<Entity>`（tech/mybatis-plus.md）。

| 文件 | 内容 | 代码定位 · 核心 Mapper 类 / 表名规律（定位用） |
|---|---|---|
| [domain/base/repository-core.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-core.md) | BaseMapper 用法 + 通用表结构 + Flyway 规范 + UID 生成 | `cn.cordys.mybatis.BaseMapper` / `cn.cordys.common.uid.IDGenerator`；通用表：operation_log、operation_log_blob、module、module_field、module_field_blob、dict、schedule、qrtz_* |
| [domain/base/repository-clue.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-clue.md) | 线索相关 Mapper + 表 | `ExtClueMapper / ExtCluePoolMapper / ExtClueCapacityMapper / ExtClueOwnerMapper` → 表前缀 crm_clue |
| [domain/base/repository-customer.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-customer.md) | 客户相关 Mapper + 表 | `ExtCustomerMapper / ExtCustomerContactMapper / ExtCustomerPoolMapper / ExtCustomerCapacityMapper / ExtCustomerOwnerMapper` → 表前缀 crm_customer / crm_contact |
| [domain/base/repository-opportunity.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-opportunity.md) | 商机相关 Mapper + 表 | `ExtOpportunityMapper / ExtOpportunityQuotationMapper / ExtOpportunityStageConfigMapper` → 表前缀 crm_opportunity / crm_quotation |
| [domain/base/repository-contract.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-contract.md) | 合同相关 Mapper + 表 | `ExtContractMapper / ExtContractInvoiceMapper / ExtContractSnapshotMapper / ExtBusinessTitleMapper` → 表前缀 crm_contract / crm_invoice / crm_payment |
| [domain/base/repository-order.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-order.md) | 订单相关 Mapper + 表 | `ExtOrderMapper / ExtOrderSnapshotMapper / ExtOrderStageConfigMapper` → 表前缀 crm_order |
| [domain/base/repository-product.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-product.md) | 产品相关 Mapper + 表 | `ExtProductMapper / ExtPriceMapper` → 表前缀 crm_product / crm_price |
| [domain/base/repository-system.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-system.md) | 系统管理 Mapper + 表（最大） | `ExtUserMapper / ExtRoleMapper / ExtDepartmentMapper / ExtModuleMapper / ExtModuleFieldMapper / ExtOperationLogMapper / ExtExportTaskMapper / ExtDictMapper / ExtScheduleMapper / ExtOrganizationConfigMapper` 等 20+ → 表前缀 sys_ |
| [domain/base/repository-approval.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-approval.md) | 审批 Mapper + 表 | `ExtApprovalFlowMapper / ExtApprovalInstanceMapper / ExtApprovalTaskMapper` → 表前缀 approval_* |
| [domain/base/repository-form.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-form.md) | 自定义表单 Mapper + 表 | `ExtCustomFormMapper / ExtCustomFormDataMapper / ExtCustomFormRoleUserMapper` → 表前缀 custom_form_* |
| [domain/base/repository-follow.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-follow.md) | 跟进 Mapper + 表 | `ExtFollowUpPlanMapper / ExtFollowUpRecordMapper / ExtCommentMapper` → 表前缀 follow_up_* / crm_comment |
| [domain/base/repository-schedule.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/domain/base/repository-schedule.md) | Quartz 调度表 + 自定义任务表 | `qrtz_*` + `sys_schedule` + Flyway 脚本 |

---

## 五、tech/ 技术规范与踩坑

| 文件 | 主题 | 何时读取 |
|---|---|---|
| [tech/security.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/security.md) | Shiro / 权限 / CSRF / API Key / SSRF | 写鉴权/登录/权限相关代码时 |
| [tech/mybatis-plus.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/mybatis-plus.md) | BaseMapper / LambdaQuery / 字段加密 | 新增查询/写 Mapper 时 |
| [tech/transaction.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/transaction.md) | 事务边界 + 异常回滚规则 | 写跨表操作 Service 时 |
| [tech/excel-io.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/excel-io.md) | FastExcel 导入导出 | 写导入/导出功能时 |
| [tech/schedule-job.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/schedule-job.md) | Quartz 定时任务规范 | 新增定时任务时 |
| [tech/redis-session.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/redis-session.md) | Session / 缓存 / 发布订阅 | 写缓存/消息相关代码时 |
| [tech/file-storage.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/file-storage.md) | Local/S3 文件引擎 | 写文件上传/下载/预览时 |
| [tech/operation-log.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/operation-log.md) | @OperationLog 规范 + SpEL | 写所有写操作接口时 |
| [tech/common-pitfalls.md](file:///e:/工作/金数湾/CordysCRM/knowledge/applications/backend/tech/common-pitfalls.md) | 后端常见踩坑清单 | 所有后端开发前快速过一遍 |

---

## 六、快速检索入口

### 按代码路径反查知识

知道代码路径时，定位知识：
```
cn.cordys.crm.{module}.controller → base/api-{module}.md
cn.cordys.crm.{module}.service    → product/{module}.md
cn.cordys.crm.{module}.domain     → base/domain-{module}.md
cn.cordys.crm.{module}.mapper     → base/repository-{module}.md
```

### 配置文件反查

| 配置需求 | 知识入口 |
|---|---|
| 权限节点 | base/api-*.md + business-rules.md 1.2 |
| 审批流触发 | product/approval.md |
| 通知事件配置 | product/system.md → message_task.json |
| 字段加解密 | tech/mybatis-plus.md |
| UID 生成 | repository-core.md |
