---
type: template
template-for: tech
status: official
updated: 2026-08-11
---

# 知识写作模板：技术规范（tech）

> 用于 applications/{app}/tech/*.md 的技术约束、框架使用规范、踩坑记录。

---

## 复制以下内容到新文件：

```markdown
---
type: tech
app: backend                    # backend / frontend-web / frontend-mobile / lib-shared / installer
tech-topic: security            # 主题：security / mybatis-plus / transaction / excel-io / schedule-job / redis-session / file-storage / operation-log / common-pitfalls
status: official
evidence: 对应实现的源码路径（至少2条）
owner: @{负责人}
verify-required: true
verify-note: API/注解名相对稳定，但属性值/版本号需核对 pom.xml/package.json
updated: YYYY-MM-DD
related:
  - knowledge/main/tech-constraints.md
---

# 技术规范：{主题中文名}

---

## 一、技术选型（为什么这样做）

- 选择的框架/库：{名字+版本}
- 没有选其他方案的原因：{1-2句话说明决策背景}
- 当前方案的边界/限制：{什么场景不适用}

---

## 二、正确用法示例（最关键，AI 直接抄）

### 场景1：{常见场景描述}

✅ **正确写法（复制即用）**：
```java
// 或 TypeScript / SQL / shell 等，对应技术主题
{正确代码示例，包含必要的注释}
```

❌ **常见错误写法（禁止）**：
```java
{错误代码示例，标注为什么错}
```

### 场景2：{另一个常见场景}

...

---

## 三、配置清单

| 配置项 | 作用 | 推荐值 / 约束 | 配置文件位置 |
|---|---|---|---|
| {key} | {说明} | | {路径} |

---

## 四、调试 & 问题排查

### 常见现象 → 排查路径

| 现象 | 第一步检查 | 第二步检查 | 根因常见原因 |
|---|---|---|---|
| {报错现象/异常现象} | | | |

### 常用调试命令 / 日志

```bash
# 例如：打开 debug 日志
logging.level.cn.cordys.xxx=DEBUG
```

---

## 五、扩展点（什么时候需要自定义）

| 扩展点 | 场景 | 接口/基类 | 是否已有实现类参考 |
|---|---|---|---|
| | | `cn.cordys.{接口名}` | 有：`{实现类路径}` |

---

## 六、相关踩坑坑位（可选）

| 坑位描述 | 触发条件 | 后果 | 规避方案 | 最早发现版本 |
|---|---|---|---|---|
| | | | | |
