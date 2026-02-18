---
name: product-thinking
description: >
  产品思维顾问技能 - 帮助用户将创意想法转化为结构化的产品构建策略,或诊断已有产品的问题并提供优化方向。
  运用七大思维工具(JTBD、逆向思维、减法思维、故事思维、灵魂三问、场景应用、问题发现)进行系统性分析。
  采用 5 阶段渐进式工作流,支持多执行模式(multi-agent/codex-assisted/sequential/plan-only),每个阶段生成独立文档,支持状态恢复。
---

# 产品思维顾问 Skill

## 触发方式

当用户说类似以下内容时触发：

- "我想做一个 XX 产品"
- "帮我分析这个产品想法"
- "我的产品有问题，帮我诊断"
- `/product-thinking` `/product-design`
- 加 `--mode codex` 使用 codex-assisted 模式

---

## 执行模式

启动时自动检测环境，选择对应模式：

| 模式 | 检测条件 | 执行方式 |
|------|---------|---------|
| **multi-agent** | Task 工具可用 | 派生专家 Agent 并行分析 |
| **codex-assisted** ⭐ | 用户指定 `--mode codex` | 规划任务 → Codex 执行 → 评审 |
| **sequential** | 无 Task 工具，有文件读写 | 单 Agent 顺序执行 |
| **plan-only** | 无文件访问 | 仅生成分析框架 |

详细指南：[references/06-codex-assisted.md](references/06-codex-assisted.md) · [references/07-multi-agent.md](references/07-multi-agent.md)

---

## 5 阶段工作流

| 阶段 | 触发 | 产出 | 详细文件 |
|------|------|------|---------|
| 1 Kickoff | 用户描述想法/问题 | 分析框架 + `.product-thinking/` 目录 | `references/01-kickoff.md` |
| 2 Analysis | 框架确认后 | 各工具分析文档 | `references/02-analysis.md` |
| 3 Execution | 所有分析完成后自动触发 | 综合分析报告 + 产品策略 | `references/03-execution.md` |
| 4 Review | Execution 完成后自动触发 | 审核报告（评分 + 改进建议） | `references/04-review.md` |
| 5 Closing | 所有工具通过审核后自动触发 | 最终报告 | `references/05-closing.md` |

---

## 并发配置（multi-agent 模式）

- 默认并发数：3（主 Agent + 2 个专家 Agent）
- 配置范围：2-10
- 优先级：启动参数 `--concurrency N` > `project.yaml` > 默认值 3

详细规则：[references/07-multi-agent.md](references/07-multi-agent.md)

---

## 项目状态管理

详见 [references/state-management.md](references/state-management.md)

---

## 参考文档

| 文件 | 内容 |
|------|------|
| `references/01-kickoff.md` | Kickoff 阶段详细指南 |
| `references/02-analysis.md` | Analysis 阶段 + 所有分析工具 |
| `references/03-execution.md` | Execution 阶段详细指南 |
| `references/04-review.md` | Review 阶段 + 评分体系 |
| `references/05-closing.md` | Closing 阶段详细指南 |
| `references/06-codex-assisted.md` | codex-assisted 模式详细指南 |
| `references/07-multi-agent.md` | multi-agent 模式 + 并发控制 |
| `references/state-management.md` | 状态管理规则 |
| `references/faq.md` | 常见问题 + 最佳实践 |
| `references/mermaid-best-practices.md` | Mermaid 图表规范 |
