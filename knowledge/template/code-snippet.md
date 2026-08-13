---
type: template
template-for: code-snippet
status: official
updated: 2026-08-11
---

# 知识写作模板：代码片段 / 示例（code-snippet）

> 用于沉淀可复用的代码模式：例如"导出 Excel 标准写法"、"自定义表单字段校验 JS"、"批量插入性能优化模式"。
> 注意：代码片段只写模式/骨架，不写具体业务逻辑。

---

## 复制以下内容到新文件：

```markdown
---
type: code-snippet
lang: java                        # java / typescript / ts-vue / sql / shell
applies-to: backend               # backend / frontend-web / frontend-mobile / lib-shared / installer / cross-app
pattern-name: {模式名，例如：Excel 导入标准写法 / 批量插入性能模式}
status: official
evidence: 参考实现的代码路径（2-3 条）
owner: @{负责人}
verify-required: true
verify-note: 依赖库版本可能变化，代码片段仅作结构参考
updated: YYYY-MM-DD
---

# 代码模式：{模式中文名}

> 场景：{什么时候用这个模式}
> 反模式：{什么时候不能用，应该用什么替代}

---

## 一、代码骨架（核心结构，AI 直接参考）

```java
// 或对应语言
@Service
public class XxxExportService extends BaseExportService {

    // 1. 权限校验（必须放最前面）
    @CsPermission("XXX:EXPORT")
    public void export(XxxExportReq req) {
        // 2. 参数校验
        validate(req);

        // 3. 注册导出任务（大文件异步，用户在导出任务中心下载）
        Long taskId = exportTaskRegistry.submit(() -> doExport(req));

        // 4. 返回任务 ID
        return taskId;
    }

    private void doExport(XxxExportReq req) {
        // 分页查询 + 流式写（内存友好）
    }
}
```

---

## 二、关键点说明（为什么这样写）

| 代码位置 | 设计决策 | 不这样做的后果 |
|---|---|---|
| 第X行：权限注解放最前 | 保证未登录/无权限直接拦截 | 泄露数据 / 越权 |
| 第X行：分页+流式写 | 避免一次性加载 10W+ 行导致 OOM | 大导出时生产 OOM |

---

## 三、真实参考位置（回到代码读完整实现）

| 文件 | 说明 |
|---|---|
| `{路径}/{File}.java` | 真实生产代码参考 |
| `{路径}/{AnotherFile}.ts` | 另一个案例 |

---

## 四、常见错误变体

| 错误写法 | 问题 | 正确做法 |
|---|---|---|
| 不分页一次查全量 | OOM | PageHelper 分页查，每页写 |
| 不用导出任务中心直接写 response | 请求超时 / 前端断连失败 | 注册到 ExportTask，前端去导出任务中心下载 |
