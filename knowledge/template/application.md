---
type: template
template-for: application
status: official
updated: 2026-08-11
---

# 知识写作模板：应用总览（application）

> 新建 `applications/{app}/application-{app}.md` 时复制本模板，**必须填写** YAML Front Matter。
> 不要删除本模板文件。

---

## 复制以下内容到新文件：

```markdown
---
type: application                  # 固定：application
status: official                   # official（正式）/ candidate（候选）/ deprecated（废弃）
evidence: 代码路径 / PRD / 文档链接  # 本知识的证据来源，越具体越好
owner: @{负责人 GitHub 或昵称}      # 知识 owner，变更时必须 review
verify-required: true              # true=编码前必须回代码核对, false=相对稳定
verify-note: 简要说明哪些项需核对    # 可选：当 verify-required=true 时说明
updated: YYYY-MM-DD                # 最后更新日期
---

# {应用名称} · 应用总览

> 代码路径：`{绝对/相对路径}`
> 技术栈：`{核心技术栈列表}`

---

## 一、应用职责

**负责什么**：
- 职责 1
- 职责 2

**不负责什么**（明确边界，避免跨应用理解错误）：
- 不负责 XXX（交给 {其他应用}）
- 不负责 YYY

---

## 二、模块结构

```
{顶层目录}/
├── {子目录A}/    # 说明
├── {子目录B}/    # 说明
└── {入口文件}
```

模块 POM / package.json 路径：
- 父：`{路径}`
- 本：`{路径}`

依赖方向：`{appA} → {appB}`（单向，标注反向依赖禁令）

---

## 三、上下游

**上游（调用方）**：

| 调用方 | 协议 | 鉴权方式 | 主要接口 |
|---|---|---|---|
| {调用方A} | HTTP / MQ / RPC | {方式} | /api/xxx/** |

**下游（被调用方）**：

| 下游 | 协议 | 用途 | 配置入口 |
|---|---|---|---|
| {下游B} | JDBC / HTTP | 数据库/缓存/外部API | {配置 key} |

---

## 四、核心模块入口速查

| 业务模块 | 包路径 / 文件路径 | 代码核对位置 |
|---|---|---|
| {模块1} | `{路径}` | Controller + Service |
| {模块2} | `{路径}` | {关键类/文件} |

---

## 五、启动/初始化流程（如适用）

```
步骤1：
步骤2：
步骤3：
```

**启动入口**：`{类/函数路径}`

---

## 六、AI 进入本应用的推荐读取路径

```
第1步: 读本文件（application-{app}.md）
第2步: 读 domain/product/ → 主干能力
第3步: 读 domain/solution/ → 差异逻辑
第4步: 读 domain/base/ → API/Domain/Repository 入口
第5步: 读 tech/ → 规范与踩坑
第6步: 回到代码仓库，实际读源码
```
```

