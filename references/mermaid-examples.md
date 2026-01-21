# Mermaid 图示例

本文档提供适用于产品分析的各类 Mermaid 图示例，帮助在分析报告中添加可视化元素。

---

## 图表类型速查

| 图表类型 | 适用场景 | 语法关键词 |
|---------|---------|-----------|
| **流程图 Flowchart** | 用户旅程、决策流程、操作步骤 | `graph` 或 `flowchart` |
| **状态图 State Diagram** | 用户状态流转、产品生命周期 | `stateDiagram-v2` |
| **时序图 Sequence Diagram** | 用户与产品交互、系统组件协作 | `sequenceDiagram` |
| **类图 Class Diagram** | 产品模块关系、数据模型 | `classDiagram` |
| **ER 图** | 实体关系、数据结构 | `erDiagram` |
| **甘特图 Gantt Chart** | 产品路线图、开发计划 | `gantt` |
| **饼图 Pie Chart** | 用户构成、功能占比 | `pie` |
| **思维导图 Mindmap** | 头脑风暴、功能分解 | `mindmap` |

---

## 1. 流程图（Flowchart）

### 1.1 用户旅程流程图

**适用场景**：JTBD 分析、用户旅程分析

```mermaid
flowchart LR
    A[认知阶段] --> B[考虑阶段]
    B --> C[决策阶段]
    C --> D[使用阶段]
    D --> E[推荐阶段]

    A -->|内容触达| A1[社交媒体/搜索]
    B -->|比较方案| B1[竞品对比]
    C -->|购买决策| C1[价格/功能]
    D -->|使用体验| D1[核心价值]
    E -->|口碑传播| E1[分享推荐]

    style A fill:#e1f5fe
    style C fill:#fff3e0
    style D fill:#e8f5e9
```

### 1.2 产品决策流程图

**适用场景**：减法思维、MVP 功能筛选

```mermaid
flowchart TD
    A[功能提案] --> B{核心假设?}
    B -->|是| C[保留]
    B -->|否| D{可延迟?}
    D -->|是| E[延后]
    D -->|否| F[舍弃]

    C --> G{最小实现?}
    G -->|是| H[MVP 核心]
    G -->|否| I[简化方案]

    style C fill:#c8e6c9
    style E fill:#fff9c4
    style F fill:#ffcdd2
    style H fill:#b2dfdb
```

### 1.3 风险分析树（逆向思维）

**适用场景**：逆向思维（事前验尸）、风险评估

```mermaid
flowchart TD
    A[产品失败] --> B[用户层面]
    A --> C[产品层面]
    A --> D[市场层面]

    B --> B1[问题不存在]
    B --> B2[解决方案无效]
    B --> B3[使用门槛高]

    C --> C1[功能过度复杂]
    C --> C2[性能不稳定]
    C --> C3[技术债务累积]

    D --> D1[竞品降维打击]
    D --> D2[市场需求变化]
    D --> D3[获客成本过高]

    style A fill:#ffcdd2
    style B1 fill:#ffebee
    style B2 fill:#ffebee
    style B3 fill:#ffebee
```

### 1.4 场景匹配矩阵（思维导图风格）

**适用场景**：场景应用分析

```mermaid
mindmap
  root((产品场景))
    数据分析
      数据可视化
      报表生成
      趋势预测
    自动化
      工作流编排
      批量处理
      定时任务
    个人效率
      任务管理
      知识沉淀
      快捷操作
    团队协作
      实时同步
      权限管理
      审批流程
```

---

## 2. 状态图（State Diagram）

### 2.1 用户生命周期状态图

**适用场景**：用户增长分析、流失分析

```mermaid
stateDiagram-v2
    [*] --> 访客: 首次触达
    访客 --> 注册: 提供邮箱/手机
    注册 --> 激活: 完成首次关键行为
    激活 --> 留存: 持续使用
    激活 --> 流失: 7天内未回访
    留存 --> 流失: 30天内未回访
    流失 --> 召回: 运营触达
    召回 --> 留存: 重新激活
    召回 --> 流失: 无响应

    note right of 激活
        Aha Moment
        关键转化节点
    end note
```

### 2.2 订单/任务状态图

**适用场景**：业务流程分析、状态流转设计

```mermaid
stateDiagram-v2
    [*] --> 待处理
    待处理 --> 进行中: 开始执行
    进行中 --> 已完成: 正常结束
    进行中 --> 已取消: 用户主动取消
    进行中 --> 失败: 执行异常
    失败 --> 待处理: 重试
    失败 --> 已取消: 放弃重试
    已完成 --> [*]
    已取消 --> [*]
```

---

## 3. 时序图（Sequence Diagram）

### 3.1 用户与产品交互时序图

**适用场景**：故事思维、用户场景分析

```mermaid
sequenceDiagram
    actor User as 用户
    participant UI as 界面
    participant Core as 核心逻辑
    participant Data as 数据层

    User->>UI: 发起操作
    UI->>Core: 调用功能
    Core->>Data: 读取/写入数据
    Data-->>Core: 返回结果
    Core->>UI: 更新状态
    UI-->>User: 展示反馈

    Note over User,UI: 用户感知的响应时间
    Note over Core,Data: 系统内部处理
```

### 3.2 多角色协作时序图

**适用场景**：B2B 产品、社交产品、平台产品

```mermaid
sequenceDiagram
    actor UserA as 用户A
    actor UserB as 用户B
    participant System as 系统
    participant Notify as 通知服务

    UserA->>System: 创建内容
    System->>System: 内容审核
    System->>Notify: 触发通知
    Notify->>UserB: 推送提醒
    UserB->>System: 查看内容
    System->>UserA: 反馈互动数据
```

---

## 4. 类图（Class Diagram）

### 4.1 产品模块关系图

**适用场景**：系统架构分析、功能模块设计

```mermaid
classDiagram
    class 用户模块 {
        +注册()
        +登录()
        +个人资料()
    }

    class 内容模块 {
        +创建()
        +编辑()
        +删除()
    }

    class 社交模块 {
        +关注()
        +点赞()
        +评论()
    }

    class 通知模块 {
        +推送()
        +消息列表()
    }

    内容模块 ..> 用户模块: 创建者
    社交模块 ..> 内容模块: 互动对象
    通知模块 ..> 社交模块: 触发通知
```

### 4.2 数据模型关系图

**适用场景**：数据结构设计、实体关系分析

```mermaid
erDiagram
    USER ||--o{ ORDER : places
    USER {
        string id PK
        string name
        string email
        datetime created_at
    }
    ORDER ||--|{ ORDER_ITEM : contains
    ORDER {
        string id PK
        string user_id FK
        string status
        float total_amount
    }
    ORDER_ITEM {
        string id PK
        string order_id FK
        string product_id FK
        int quantity
    }
    PRODUCT ||--o{ ORDER_ITEM : ""
    PRODUCT {
        string id PK
        string name
        float price
        int stock
    }
```

---

## 5. 甘特图（Gantt Chart）

### 5.1 产品路线图

**适用场景**：产品规划、里程碑设定

```mermaid
gantt
    title 产品开发路线图
    dateFormat YYYY-MM-DD
    section 市场验证
    用户访谈       :done, m1, 2024-01-01, 15d
    需求分析       :done, m2, after m1, 10d
    section MVP 开发
    核心功能开发   :active, m3, 2024-01-26, 30d
    内部测试       :m4, after m3, 10d
    section 上线准备
    小范围公测     :m5, after m4, 15d
    正式发布       :m6, after m5, 5d
```

---

## 6. 饼图（Pie Chart）

### 6.1 用户构成分析

**适用场景**：用户画像分析、市场定位

```mermaid
pie title 用户年龄分布
    "18-24岁" : 15
    "25-34岁" : 45
    "35-44岁" : 25
    "45-54岁" : 10
    "55岁+" : 5
```

### 6.2 功能使用占比

**适用场景**：功能优先级判断、资源分配

```mermaid
pie title 功能使用频率分布
    "核心功能" : 60
    "次要功能" : 25
    "辅助功能" : 10
    "未使用" : 5
```

---

## 7. 综合示例：完整产品分析

### 7.1 灵魂三问可视化

```mermaid
flowchart LR
    subgraph Q1[谁是用户？]
        A1[目标人群]
        A2[使用场景]
        A3[核心需求]
    end

    subgraph Q2[痛点是什么？]
        B1[当前解决方案]
        B2[存在的问题]
        B3[改进空间]
    end

    subgraph Q3[为什么是你？]
        C1[独特资源]
        C2[技术优势]
        C3[市场机会]
    end

    Q1 --> D{价值验证}
    Q2 --> D
    Q3 --> D

    D -->|通过| E[✅ 值得深入]
    D -->|未通过| F[❌ 需要调整]

    style Q1 fill:#e3f2fd
    style Q2 fill:#fff3e0
    style Q3 fill:#e8f5e9
    style E fill:#c8e6c9
    style F fill:#ffcdd2
```

### 7.2 JTBD 三层需求分析

```mermaid
mindmap
  root((JTBD))
    功能层
      完成具体任务
      解决实际问题
      提升效率
    情感层
      减少焦虑
      增强信心
      获得成就感
    社会层
      身份认同
      社交货币
      归属感
```

---

## 使用建议

### 何时使用流程图
- 用户旅程分析
- 决策路径梳理
- 风险因素分解
- 功能筛选逻辑

### 何时使用状态图
- 用户生命周期分析
- 业务状态流转设计
- 订单/任务流程
- 用户增长漏斗

### 何时使用时序图
- 用户交互场景描述
- 系统协作关系
- 多角色业务流程
- API 交互设计

### 何时使用类图/ER 图
- 产品模块设计
- 数据模型规划
- 系统架构梳理
- 功能边界划分

### 何时使用思维导图
- 头脑风暴
- 功能分解
- 场景分类
- 竞品分析

---

## 最佳实践

1. **简洁优先**：不要过度复杂化，能用简单图表表达就不用复杂的
2. **突出重点**：用颜色、样式突出关键节点
3. **层次清晰**：复杂流程可以拆分成多个图表
4. **保持一致**：同一份报告中使用统一的图表风格
5. **文字配合**：图表要有标题和必要的说明文字
