# 前端共享包（lib-shared）· 索引（INDEX）

> 进入 @lib/shared 后的导航。

---

## 一、高频 API 模块速查

| 模块文件 | 对应业务 | 典型函数 |
|---|---|---|
| [api/modules/clue.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/clue.ts) | 线索 | list / detail / add / update / convert / pool 操作 |
| [api/modules/customer.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/customer.ts) | 客户 | customer + contact + pool + collaboration |
| [api/modules/opportunity.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/opportunity.ts) | 商机 | opportunity + quotation + stage |
| [api/modules/contract.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/contract.ts) | 合同 | contract + invoice + payment plan + payment record |
| [api/modules/order.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/order.ts) | 订单 | order + stage |
| [api/modules/product.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/product.ts) | 产品 | product + price + field |
| [api/modules/follow.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/follow.ts) | 跟进 | plan + record + comment |
| [api/modules/dashboard.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/dashboard.ts) | 仪表盘 | dashboard + collection + module |
| [api/modules/home.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/home.ts) | 首页统计 | home overview |
| [api/modules/customForm.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/customForm.ts) | 自定义表单 | form design + form data |
| [api/modules/system/{login,org,role,module,log}](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/system/) | 系统管理 | 登录/组织/角色/模块/日志 |
| [api/modules/agent.ts + ai.ts](file:///e:/工作/金数湾/CordysCRM/frontend/packages/lib-shared/api/modules/) | 智能体/AI | agent 配置 + AI skill 调用 |

---

## 二、枚举速查

| 枚举文件 | 内容 |
|---|---|
| `enums/clueEnum.ts` | ClueStatus（NEW/FOLLOWING/INTERESTED） |
| `enums/customerEnum.ts` | CustomerSearchType / FollowPlanStatus |
| `enums/opportunityEnum.ts` | OpportunitySearchType / CirculationType |
| `enums/contractEnum.ts` | ContractStatus / PaymentPlan / BusinessTitleStatus |
| `enums/formDesignEnum.ts` | 表单设计组件类型 |
| `enums/httpEnum.ts` | HTTP 状态码 / Content-Type / Header Key |
| `enums/moduleEnum.ts` | 模块 key 与名称映射 |
| `enums/tableEnum.ts` | 表格列类型 / 筛选类型 |
| `enums/process.ts` | 审批流节点类型 / 审批人类型 |
| `enums/systemEnum.ts` | 系统级枚举 |
| `enums/commonEnum.ts` | YES/NO / 开关 / 性别 / 通用状态 |
| `enums/formula.ts` | 公式类型 |
| `enums/uploadEnum.ts` | 上传类型 / 大小限制 |

---

## 三、与后端 API 的对照

前端 `api/requrls/{module}/index.ts` 中的 URL 常量 → 后端 `cn.cordys.crm.{module}.controller` 中 `@RequestMapping` 路径一一对应。
