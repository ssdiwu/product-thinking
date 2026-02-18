# 状态管理详细说明

## 目录
- [为什么需要状态管理](#为什么需要状态管理)
- [project.yaml 结构说明](#projectyaml-结构说明)
- [状态恢复流程](#状态恢复流程)
- [状态更新时机](#状态更新时机)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

---

## 为什么需要状态管理

### AI 的技术限制

**核心事实**: AI 没有持久状态

- 每次对话结束,所有内部状态都消失
- KV Cache 只是计算加速,不是记忆
- 对话重启后,AI 无法自动恢复之前的上下文

### 产品思考的特点

**多阶段、长周期**:
- 5 个阶段: Kickoff → Analysis → Execution → Review → Closing
- 每个阶段可能需要多轮对话
- 用户可能在任何时候中断对话

**决策密集**:
- 灵魂三问的回答
- 分析工具的选择
- 产品策略的制定
- 风险评估的结果

**文档依赖**:
- 每个阶段生成独立文档
- 后续阶段依赖前面阶段的输出
- 需要追踪哪些文档已生成、哪些待生成

### 状态管理的价值

**支持对话恢复**:
- 用户说"继续之前的产品分析"
- AI 可以读取 project.yaml 了解当前进度
- 从中断点继续,而非重新开始

**保持决策一致性**:
- 已确认的决策不会被遗忘
- 避免重复讨论已解决的问题
- 确保整个分析过程的连贯性

**提供进度可见性**:
- 用户可以随时查看当前进度
- 清楚知道已完成什么、待完成什么
- 便于项目管理和时间规划

---

## project.yaml 结构说明

### 完整结构

```yaml
project_name: "产品名称"
created_at: "2026-02-12T00:00:00Z"
updated_at: "2026-02-12T00:00:00Z"
phase: "kickoff"  # 当前阶段: kickoff/analysis/execution/review/closing

# 灵魂三问
soul_questions:
  who: "目标用户是谁"
  pain: "核心痛点是什么"
  why: "为什么选择你"

# 客户画像(B2B 场景)
customer_profile:
  company_size: "公司规模"
  industry: "行业"
  decision_maker: "决策者"
  budget_range: "预算范围"
  procurement_cycle: "采购周期"

# 选定的分析工具
selected_tools:
  - jtbd          # Jobs-to-be-Done 分析
  - mvp           # MVP 功能审视
  - scenarios     # 场景应用分析
  - diagnosis     # 问题发现诊断

# 各阶段状态
phases:
  kickoff:
    status: "completed"
    completed_at: "2026-02-12T00:00:00Z"
  analysis:
    status: "in_progress"
    started_at: "2026-02-12T00:00:00Z"
  execution:
    status: "pending"
  review:
    status: "pending"
  closing:
    status: "pending"
```

---

### 字段说明

#### 1. 项目元数据

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `project_name` | string | 是 | 产品名称,用于标识项目 |
| `created_at` | datetime | 是 | 项目创建时间,ISO 8601 格式 |
| `updated_at` | datetime | 是 | 最后更新时间,每次修改都要更新 |
| `phase` | enum | 是 | 当前阶段: kickoff/analysis/execution/review/closing |

**phase 字段的含义**:
- `kickoff`: 正在进行 Kickoff 阶段
- `analysis`: 正在进行 Analysis 阶段
- `execution`: 正在进行 Execution 阶段
- `review`: 正在进行 Review 阶段
- `closing`: 正在进行 Closing 阶段

---

#### 2. 灵魂三问

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `soul_questions.who` | string | 是 | 目标用户是谁 |
| `soul_questions.pain` | string | 是 | 核心痛点是什么 |
| `soul_questions.why` | string | 是 | 为什么选择你 |

**填充时机**: Kickoff 阶段

**示例**:
```yaml
soul_questions:
  who: "25-35 岁的互联网从业者,需要频繁切换多个工作账号"
  pain: "每次切换账号需要重新登录,平均耗时 30 秒,每天切换 20+ 次"
  why: "一键切换,无需重新登录,支持指纹/面容识别"
```

---

#### 3. 客户画像(B2B 场景)

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `customer_profile.company_size` | string | 否 | 公司规模(如: 50-200人) |
| `customer_profile.industry` | string | 否 | 行业(如: SaaS、制造业) |
| `customer_profile.decision_maker` | string | 否 | 决策者(如: CTO、采购经理) |
| `customer_profile.budget_range` | string | 否 | 预算范围(如: 10-50万/年) |
| `customer_profile.procurement_cycle` | string | 否 | 采购周期(如: 3-6个月) |

**填充时机**: Kickoff 阶段(仅 B2B 场景)

**示例**:
```yaml
customer_profile:
  company_size: "50-200人"
  industry: "SaaS"
  decision_maker: "CTO"
  budget_range: "10-50万/年"
  procurement_cycle: "3-6个月"
```

---

#### 4. 选定的分析工具

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `selected_tools` | array | 是 | 选定的分析工具列表 |

**可选值**:
- `jtbd`: Jobs-to-be-Done 分析
- `mvp`: MVP 功能审视
- `scenarios`: 场景应用分析
- `diagnosis`: 问题发现诊断

**填充时机**: Kickoff 阶段

**示例**:
```yaml
selected_tools:
  - jtbd
  - mvp
  - scenarios
```

---

#### 5. 各阶段状态

每个阶段都有以下字段:

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `status` | enum | 是 | 阶段状态: pending/in_progress/completed |
| `started_at` | datetime | 否 | 开始时间(status=in_progress 时填充) |
| `completed_at` | datetime | 否 | 完成时间(status=completed 时填充) |

**status 字段的含义**:
- `pending`: 待开始
- `in_progress`: 进行中
- `completed`: 已完成

**示例**:
```yaml
phases:
  kickoff:
    status: "completed"
    completed_at: "2026-02-12T00:00:00Z"
  analysis:
    status: "in_progress"
    started_at: "2026-02-12T00:10:00Z"
  execution:
    status: "pending"
  review:
    status: "pending"
  closing:
    status: "pending"
```

---

## 状态恢复流程

### 对话中断场景

**场景 1: 用户主动中断**
```
用户: 我想做一个 XX 产品
AI: 让我用 product-thinking 框架帮你分析...
    (完成 Kickoff 阶段,开始 Analysis 阶段)
用户: 我先去忙别的,晚点再继续
    (对话结束)
```

**场景 2: 系统限制**
```
用户: 继续分析
AI: (执行 JTBD 分析)
    (上下文即将用完,对话被压缩)
```

---

### 恢复流程

```mermaid
graph TD
    A[用户说"继续"] --> B[读取 project.yaml]
    B --> C{project.yaml<br/>是否存在?}

    C -->|否| D[询问用户项目名称]
    D --> E[搜索对应的 project.yaml]
    E --> F{找到?}
    F -->|否| G[提示用户项目不存在]
    F -->|是| H[读取 project.yaml]

    C -->|是| H
    H --> I[解析当前阶段和状态]
    I --> J[读取对应阶段的文档]
    J --> K[向用户确认上下文]
    K --> L[从中断点继续]

    style A fill:#e3f2fd
    style L fill:#c8e6c9
    style G fill:#ffcdd2
```

---

### 恢复步骤

#### 1. 读取 project.yaml

```bash
# 使用 Read 工具读取
Read .product-thinking/project.yaml
```

#### 2. 解析当前状态

从 project.yaml 中提取:
- `phase`: 当前阶段
- `phases.<phase>.status`: 当前阶段的状态
- `selected_tools`: 选定的分析工具
- `soul_questions`: 灵魂三问的回答

#### 3. 读取对应阶段的文档

根据当前阶段,读取对应的文档:

| 阶段 | 文档路径 |
|------|---------|
| kickoff | `phases/01-kickoff.md` |
| analysis | `phases/02-jtbd.md`, `phases/03-mvp.md`, ... |
| execution | `analysis/综合分析.md` |
| review | `reviews/completeness-check.md` |
| closing | `reports/final-report.md` |

#### 4. 向用户确认上下文

**确认话术**:
```
我看到你之前在做 [产品名称] 的产品分析,当前进度:

- **当前阶段**: [阶段名称]
- **已完成**: [已完成的工作]
- **下一步**: [下一步要做的工作]

让我们继续...
```

**示例**:
```
我看到你之前在做"账号快速切换工具"的产品分析,当前进度:

- **当前阶段**: Analysis (分析阶段)
- **已完成**:
  - Kickoff 阶段: 灵魂三问已回答,选定了 JTBD、MVP、场景三个分析工具
  - JTBD 分析: 已完成,核心发现见 phases/02-jtbd.md
- **下一步**: 执行 MVP 功能审视

让我们继续 MVP 功能审视...
```

#### 5. 从中断点继续

根据当前阶段和状态,继续执行:

| 阶段 | 状态 | 下一步行动 |
|------|------|-----------|
| kickoff | in_progress | 继续提问灵魂三问或选择分析工具 |
| kickoff | completed | 进入 Analysis 阶段 |
| analysis | in_progress | 继续执行当前分析工具或开始下一个 |
| analysis | completed | 进入 Execution 阶段 |
| execution | in_progress | 继续生成综合分析报告 |
| execution | completed | 进入 Review 阶段 |
| review | in_progress | 继续完整性检查 |
| review | completed | 进入 Closing 阶段 |
| closing | in_progress | 继续生成最终报告 |
| closing | completed | 项目完成 |

---

## 状态更新时机

### 关键时机

**1. 项目初始化**
- 时机: 用户开始新的产品分析
- 操作: 创建 `.product-thinking/` 目录和 `project.yaml`
- 字段: 填充 `project_name`, `created_at`, `updated_at`, `phase=kickoff`

**2. 完成灵魂三问**
- 时机: 用户回答完三个灵魂问题
- 操作: 更新 `soul_questions` 字段
- 字段: `soul_questions.who`, `soul_questions.pain`, `soul_questions.why`

**3. 选定分析工具**
- 时机: 根据灵魂三问选择分析工具
- 操作: 更新 `selected_tools` 字段
- 字段: `selected_tools` 数组

**4. 完成 Kickoff 阶段**
- 时机: 用户确认分析框架
- 操作: 更新 Kickoff 阶段状态,切换到 Analysis 阶段
- 字段:
  - `phases.kickoff.status = "completed"`
  - `phases.kickoff.completed_at = "当前时间"`
  - `phases.analysis.status = "in_progress"`
  - `phases.analysis.started_at = "当前时间"`
  - `phase = "analysis"`
  - `updated_at = "当前时间"`

**5. 完成每个分析工具**
- 时机: 用户确认某个分析工具的输出
- 操作: 记录在对应的阶段文档中(不更新 project.yaml)

**6. 完成 Analysis 阶段**
- 时机: 所有选定的分析工具都已执行完毕
- 操作: 更新 Analysis 阶段状态,切换到 Execution 阶段
- 字段:
  - `phases.analysis.status = "completed"`
  - `phases.analysis.completed_at = "当前时间"`
  - `phases.execution.status = "in_progress"`
  - `phases.execution.started_at = "当前时间"`
  - `phase = "execution"`
  - `updated_at = "当前时间"`

**7. 完成 Execution 阶段**
- 时机: 用户确认综合分析报告
- 操作: 更新 Execution 阶段状态,切换到 Review 阶段
- 字段:
  - `phases.execution.status = "completed"`
  - `phases.execution.completed_at = "当前时间"`
  - `phases.review.status = "in_progress"`
  - `phases.review.started_at = "当前时间"`
  - `phase = "review"`
  - `updated_at = "当前时间"`

**8. 完成 Review 阶段**
- 时机: 综合评分 >= 80 分,用户确认审核结果
- 操作: 更新 Review 阶段状态,切换到 Closing 阶段
- 字段:
  - `phases.review.status = "completed"`
  - `phases.review.completed_at = "当前时间"`
  - `phases.closing.status = "in_progress"`
  - `phases.closing.started_at = "当前时间"`
  - `phase = "closing"`
  - `updated_at = "当前时间"`

**9. 完成 Closing 阶段**
- 时机: 用户确认最终报告
- 操作: 更新 Closing 阶段状态,项目完成
- 字段:
  - `phases.closing.status = "completed"`
  - `phases.closing.completed_at = "当前时间"`
  - `phase = "completed"`
  - `updated_at = "当前时间"`

---

### 更新原则

**1. 及时更新**
- 每次阶段切换都要立即更新 project.yaml
- 不要等到对话结束才更新

**2. 原子更新**
- 每次只更新相关的字段
- 不要一次性修改大量无关字段

**3. 保持一致性**
- `phase` 字段必须与 `phases.<phase>.status` 一致
- `updated_at` 必须在每次修改时更新

**4. 记录时间**
- 所有时间字段使用 ISO 8601 格式
- 使用 UTC 时区或明确标注时区

---

## 最佳实践

### 1. 定期保存状态

**在这些时刻保存状态**:
- 完成一个阶段后
- 做出重要决策后
- 对话即将结束前
- 用户明确表示要暂停时

**示例**:
```
用户: 我先去忙别的,晚点再继续
AI: 好的,我已经保存了当前进度:
    - 当前阶段: Analysis
    - 已完成: JTBD 分析
    - 下一步: MVP 功能审视

    下次说"继续"就可以从这里开始。
```

---

### 2. 状态快照

**什么是状态快照**:
在关键节点创建一个完整的状态记录,包含:
- 当前进度
- 已确认的决策
- 当前假设
- 开放问题

**状态快照模板**:
```markdown
## 状态快照 [日期]

### 当前进度
- 阶段: [当前阶段]
- 状态: [进行中的工作]

### 已确认决策
- 决策1: [决策内容] (理由: [理由])
- 决策2: [决策内容] (理由: [理由])

### 当前假设
- 假设1: [假设内容] (待验证)
- 假设2: [假设内容] (待验证)

### 开放问题
- 问题1: [问题描述] (优先级: 高/中/低)
- 问题2: [问题描述] (优先级: 高/中/低)
```

**保存位置**: 在对应阶段的文档末尾添加

---

### 3. 版本控制

**使用 Git 管理状态**:
```bash
# 初始化 Git 仓库
cd .product-thinking
git init

# 每次阶段切换后提交
git add .
git commit -m "完成 Kickoff 阶段"

# 查看历史
git log --oneline
```

**好处**:
- 可以回溯到任何历史状态
- 可以对比不同版本的差异
- 可以创建分支探索不同方案

---

### 4. 备份重要决策

**在多个地方记录关键决策**:
1. `project.yaml` - 结构化数据
2. 阶段文档 - 详细说明
3. 状态快照 - 完整上下文

**示例**:
```yaml
# project.yaml
soul_questions:
  who: "25-35 岁的互联网从业者"
  pain: "账号切换耗时 30 秒,每天 20+ 次"
  why: "一键切换,无需重新登录"
```

```markdown
# phases/01-kickoff.md
## 灵魂三问

### 1. 谁是你的目标用户?
25-35 岁的互联网从业者,需要频繁切换多个工作账号。

**具体画像**:
- 职业: 产品经理、设计师、开发者
- 场景: 需要同时管理多个客户/项目的账号
- 痛点: 每次切换都要重新登录,打断工作流
```

---

### 5. 清晰的命名

**project.yaml 的命名**:
- 使用产品名称作为 `project_name`
- 避免使用"项目1"、"测试"等模糊名称

**示例**:
```yaml
# ❌ 不好的命名
project_name: "项目1"

# ✅ 好的命名
project_name: "账号快速切换工具"
```

---

## 常见问题

### Q: 如果 project.yaml 丢失了怎么办?

**A**: 可以从阶段文档中重建:
1. 读取所有阶段文档
2. 提取关键信息(灵魂三问、选定工具等)
3. 根据文档的存在情况判断当前阶段
4. 重新创建 project.yaml

**重建脚本**:
```bash
# 检查哪些阶段文档存在
ls -la phases/
ls -la analysis/
ls -la reviews/
ls -la reports/

# 根据存在的文档推断当前阶段
```

---

### Q: 如何处理多个产品分析项目?

**A**: 为每个项目创建独立的目录:
```
~/projects/
├── product-a/
│   └── .product-thinking/
│       ├── project.yaml
│       ├── phases/
│       ├── analysis/
│       ├── reviews/
│       └── reports/
└── product-b/
    └── .product-thinking/
        ├── project.yaml
        ├── phases/
        ├── analysis/
        ├── reviews/
        └── reports/
```

**切换项目**:
```bash
cd ~/projects/product-a
# 继续 product-a 的分析

cd ~/projects/product-b
# 继续 product-b 的分析
```

---

### Q: 如何在团队中共享状态?

**A**: 使用 Git 仓库:
```bash
# 初始化 Git 仓库
cd .product-thinking
git init
git add .
git commit -m "初始化产品分析项目"

# 推送到远程仓库
git remote add origin <远程仓库地址>
git push -u origin main

# 团队成员克隆
git clone <远程仓库地址>
```

**协作流程**:
1. 每次完成一个阶段后提交
2. 推送到远程仓库
3. 团队成员拉取最新状态
4. 继续下一个阶段

---

### Q: 状态文件会不会很大?

**A**: 不会。project.yaml 通常只有几 KB:
- 只包含元数据和状态信息
- 不包含详细的分析内容(在阶段文档中)
- 结构简单,易于解析

**示例大小**:
```bash
$ ls -lh .product-thinking/project.yaml
-rw-r--r--  1 user  staff   2.1K  2026-02-12 00:00 project.yaml
```

---

### Q: 如何验证 project.yaml 的正确性?

**A**: 检查以下几点:
1. **YAML 语法正确**: 使用 YAML 验证工具
2. **必填字段存在**: `project_name`, `created_at`, `updated_at`, `phase`
3. **时间格式正确**: ISO 8601 格式
4. **阶段状态一致**: `phase` 与 `phases.<phase>.status` 一致
5. **时间逻辑正确**: `started_at` < `completed_at`

**验证脚本**:
```bash
# 使用 yq 验证 YAML 语法
yq eval '.product-thinking/project.yaml'

# 检查必填字段
yq eval '.project_name' .product-thinking/project.yaml
yq eval '.phase' .product-thinking/project.yaml
```

---

### Q: 对话压缩后状态会丢失吗?

**A**: 不会。状态保存在文件系统中:
- `project.yaml` 保存在 `.product-thinking/` 目录
- 对话压缩只影响 AI 的内存,不影响文件
- 下次对话时重新读取 `project.yaml` 即可恢复

**关键原则**: 状态持久化到文件,而非依赖 AI 的内存

---

### Q: 如何处理状态冲突?

**A**: 如果多人同时修改 project.yaml:
1. 使用 Git 的合并功能
2. 手动解决冲突
3. 确保 `updated_at` 字段是最新的

**避免冲突**:
- 明确分工,避免同时修改同一阶段
- 使用 Git 分支进行并行工作
- 定期同步状态

---

**维护者**: 507
**创建时间**: 2026-02-12T01:35:00Z
**基于**: ai-team/references/state-management.md
