# 并发数配置指南

> 适用于 multi-agent 模式

## 为什么需要配置并发数？

不同用户的 API 速率限制不同：
- **Claude Pro 用户**：建议 3-5 个并发
- **企业用户**（更高速率限制）：可以设置 5-10 个并发
- **测试环境**：建议 2 个并发（最小值）

合理配置并发数可以：
- ✅ 加速分析任务执行（更多专家 Agent 并行工作）
- ✅ 避免触发 429 错误（速率限制）
- ✅ 充分利用 API 配额

---

## 配置方式

### 方式 1: 启动参数（推荐）

**优先级最高**，适合临时调整或测试不同并发数。

```bash
# 使用 5 个并发
/product-design --concurrency 5

# 使用最小并发（测试环境）
/product-design --concurrency 2

# 使用最大并发（企业用户）
/product-design --concurrency 10
```

### 方式 2: project.yaml 配置

**适合固定项目配置**，每次启动自动使用。

```yaml
# .product-thinking/project.yaml
project_name: "智能客服系统产品设计"
concurrency: 5  # 配置并发数为 5
analysis_tools:
  - jtbd
  - mvp
  - scenarios
  - diagnosis
```

### 方式 3: 默认值

**无需配置**，向后兼容。

```
默认值: 3（主 Agent + 2 个专家 Agent）
```

---

## 配置验证规则

| 配置值 | 验证结果 | 说明 |
|-------|---------|------|
| `< 2` | ❌ 使用默认值 3 | 最小值为 2（主 Agent + 1 个专家 Agent） |
| `2-10` | ✅ 使用配置值 | 有效范围 |
| `> 10` | ❌ 使用默认值 3 | 最大值为 10（避免过度并发） |
| 非数字 | ❌ 使用默认值 3 | 无效配置 |

**示例**：

```bash
# 配置值为 1（小于最小值）
/product-design --concurrency 1
# 输出: ⚠️ 启动参数中的并发数 1 小于最小值 2，使用默认值 3

# 配置值为 15（大于最大值）
/product-design --concurrency 15
# 输出: ⚠️ 启动参数中的并发数 15 大于最大值 10，使用默认值 3

# 配置值为 5（有效）
/product-design --concurrency 5
# 输出: ℹ️ 使用启动参数中的并发数配置: 5
```

---

## 配置优先级

当多个配置同时存在时，按以下优先级选择：

```
启动参数 > project.yaml > 默认值
```

**示例**：

```yaml
# project.yaml 中配置为 3
concurrency: 3
```

```bash
# 启动时指定为 5
/product-design --concurrency 5

# 实际使用: 5（启动参数优先级更高）
```

---

## 并发数对执行的影响

### 示例 1: 并发数 = 3（默认）

```
输入: tasks = [JTBD, MVP, 场景, 诊断]
配置: concurrency = 3

计算:
  可用专家 Agent 槽位 = 3 - 1 = 2
  批次数 = ceil(4 / 2) = 2

执行:
  批次 1: [JTBD, MVP] → 并行执行 → 等待全部完成
  批次 2: [场景, 诊断] → 并行执行 → 等待全部完成

  product-thinking 在每个批次之间做结果汇总和质量检查
```

**执行时间**：约 2 个批次的时间

---

### 示例 2: 并发数 = 5

```
输入: tasks = [JTBD, MVP, 场景, 诊断]
配置: concurrency = 5

计算:
  可用专家 Agent 槽位 = 5 - 1 = 4
  批次数 = ceil(4 / 4) = 1

执行:
  批次 1: [JTBD, MVP, 场景, 诊断] → 并行执行 → 等待全部完成

  product-thinking 在批次完成后做结果汇总和质量检查
```

**执行时间**：约 1 个批次的时间（速度提升 50%）

---

### 示例 3: 并发数 = 2（最小值）

```
输入: tasks = [JTBD, MVP, 场景, 诊断]
配置: concurrency = 2

计算:
  可用专家 Agent 槽位 = 2 - 1 = 1
  批次数 = ceil(4 / 1) = 4

执行:
  批次 1: [JTBD] → 执行 → 等待完成
  批次 2: [MVP] → 执行 → 等待完成
  批次 3: [场景] → 执行 → 等待完成
  批次 4: [诊断] → 执行 → 等待完成

  product-thinking 在每个批次之间做结果汇总和质量检查
```

**执行时间**：约 4 个批次的时间（接近串行模式）

---

## 配置建议

### Claude Pro 用户

**推荐配置**: 3-5

```bash
# 保守配置（稳定性优先）
/product-design --concurrency 3

# 激进配置（速度优先）
/product-design --concurrency 5
```

**理由**：
- Claude Pro 的 API 速率限制大约支持 3-5 个并发会话
- 超过 5 个可能频繁触发 429 错误

---

### 企业用户（更高速率限制）

**推荐配置**: 5-10

```bash
# 标准配置
/product-design --concurrency 7

# 最大配置（大型项目）
/product-design --concurrency 10
```

**理由**：
- 企业用户通常有更高的 API 速率限制
- 可以充分利用并发能力加速分析

---

### 测试环境

**推荐配置**: 2

```bash
# 最小并发（测试环境）
/product-design --concurrency 2
```

**理由**：
- 测试环境通常不需要高速执行
- 最小并发可以节省 API 配额
- 便于调试和观察执行流程

---

## 常见问题

### Q: 如何知道我的 API 速率限制是多少？

**A**: 可以通过以下方式测试：

1. 从默认值 3 开始
2. 如果频繁触发 429 错误 → 降低到 2
3. 如果从未触发 429 错误 → 尝试提高到 5
4. 逐步调整直到找到最佳值

---

### Q: 并发数越高越好吗？

**A**: 不一定。并发数过高会导致：

- ❌ 频繁触发 429 错误（速率限制）
- ❌ 降级为串行模式（反而更慢）
- ❌ 浪费 API 配额

**建议**：根据实际 API 速率限制选择合适的并发数。

---

### Q: 如果触发 429 错误会怎样？

**A**: product-thinking 会自动降级：

```
正常模式（并发数 N）
    ↓ 触发 429
降级模式（并发数 1）
    ↓ 持续 429
串行模式（主 Agent 独自完成）
```

**关键**：降级后任务仍会完成，只是速度变慢。

---

### Q: 可以在执行过程中动态调整并发数吗？

**A**: 不可以。并发数在 Kickoff 阶段确定后不可更改。

如果需要调整：
1. 停止当前执行
2. 修改配置
3. 重新启动

---

## 配置检测逻辑（给 AI 读）

在 Kickoff 阶段，按以下逻辑检测并发数：

```python
# 伪代码
def detect_concurrency():
    # 1. 检查启动参数（最高优先级）
    if args.concurrency:
        return validate_concurrency(args.concurrency, "启动参数")

    # 2. 检查 project.yaml
    if project_yaml.concurrency:
        return validate_concurrency(project_yaml.concurrency, "project.yaml")

    # 3. 使用默认值
    return 3

def validate_concurrency(value, source):
    if value < 2:
        warn(f"{source} 中的并发数 {value} 小于最小值 2，使用默认值 3")
        return 3
    if value > 10:
        warn(f"{source} 中的并发数 {value} 大于最大值 10，使用默认值 3")
        return 3
    info(f"使用 {source} 中的并发数配置: {value}")
    return value
```

**关键约束**：
- 最小值：2（主 Agent + 1 个专家 Agent）
- 最大值：10（避免过度并发）
- 无效值：使用默认值 3 并警告用户

---

**维护者**: 507
**最后更新**: 2026-02-12
**相关文档**:
- [07-multi-agent.md](./07-multi-agent.md) - multi-agent 模式详细指南
- [~/.claude/rules/common/concurrent-control.md](~/.claude/rules/common/concurrent-control.md) - 并发控制规则
