# 双脑系统 (Dual-Brain System) 规范文档 (V2.1) 💡

📖 **中文版本** | [English Version](README.md)

*为了优化 Side Project 的产出比，设计的一套基于元认知理论的 AI 协作协议 🤖*

**最新更新 (v2.1)**:
- 🔗 依赖关系图分析（自动检测文件关系）
- 📖 参考范围（Coder 只读上下文）
- ⚡ 调试模式（简单错误的快速修复路径 E→C）
- 🔍 影响审计（API 副作用追踪）

---

## 目录

- [简介](#简介)
- [存储架构](#一-存储架构-memory-structure)
- [角色与权限](#二-角色与权限-agent-roles--permissions)
- [标准化作业流程](#三-标准化作业流程-sop)
- [新功能 (v2.1)](#四-新功能-v21)
- [防御性规则](#五-防御性规则)
- [按需分配模型](#六-按需分配模型-cost-aware-routing)
- [快速开始指南](#七-初始化与快速开始指南)
- [工作流示例](#具体示例构建你的第一个功能)

---

# 简介

关于这个理念的落地，我结合了之前开会时 Hiredly InnoTech 团队 CTO 和我提及的"规格驱动开发（Spec-Driven Development）"概念，并参考了之前读过的丹尼尔·卡尼曼在《思考，快与慢》中提出的系统1与系统2理论，创建了一个基于 Markdown 的文件系统。书中定义的系统1是快速、直觉且感性的，负责日常的自动化决策；而系统2则是缓慢、审慎且逻辑化的，负责复杂的分析性推理。我设计的这套系统是在模拟这种认知运作方式，将"规格"作为引导 AI **系统2深度思考的媒介**并把任务细分为能够为**系统1的直觉快速处理常规任务**，更能通过规格文档的约束进入系统2的慢思考模式，对复杂的业务逻辑进行严密的推演和架构设计。

在目前的系统实践中 (v2.1)，我设计了由**长期记忆**与**短期记忆**组成的规格结构。长期记忆涵盖了项目结构、数据库关系文档、软件需求说明（SRS）、服务端动作逻辑以及**依赖关系**，构成了项目的核心知识底座；短期记忆则通过 `_PLAN.md` 文件来定义具体任务的行动方案，明确目标、范围及验证清单，从而形成任务的初步全局视图。随后，系统会基于计划生成 `_INSTRUCTION.md`，将构思转化为一系列可执行的指令流。这种将元认知思维引入项目结构的做法，结合近期推出的"Agent-skill（代理技能）"机制，使 AI 能够脱离固定的职业标签，转而以技能和规格为驱动，实现更具灵活性和自主性的深度协作。

其实更主要的是为了节省开发 Side Project 的成本，能够透过不同的 IDE 或者 CLI 来协作达到低成本且通用于不同模型的 Adapter 的感觉，他们只需要明白目前的任务以及结构作为输入的媒介，这就很大程度的减少了 Token 的消耗，并且规范了它们能够触及的范围，能够更加全面的掌控整体的进程，但是比较小型的 Task 就还是一样 Single File，Single Prompt 就能够解决了，不需要到杀鸡用牛刀的感觉。

---

## 一、 存储架构 (Memory Structure)

系统分为 **长期记忆 (_DOCS)** 和 **工作记忆 (_TASK)**，实现"事实"与"意图"的物理隔离。

### 1. 📂 _DOCS (长期记忆 - 静态真理)

> **原则**：AI 只读不写（除 Evaluator 记录日志与 Archivist 更新外）。

| 文件 | 描述 | 维护者 | 更新方式 |
|------|------|--------|----------|
| **00_STRUCTURE.md** | 文件树地图。防止 AI 虚构路径。 | 自动生成 | `npm run gen:structure` |
| **01_DB_SCHEMA.md** | 数据库模式。字段、类型、关系的唯一真理来源。 | 手动 | 人工编辑 |
| **02_STYLE_GUIDE.md** | UI/UX 规范。确保视觉一致性。 | 手动 | 人工编辑 |
| **03_SERVER_ACTIONS.md** | API 协议 + 副作用。定义前后端契约。 | 手动 + Archivist | 人工 + 影响审计 |
| **04_TECH_STACK.md** | 官方允许的技术栈和版本。 | 手动 | 人工编辑 |
| **05_PROJECT_SNAPSHOT.md** | 高密度项目状态。当前功能、决策、债务。 | Archivist | 里程碑后 |
| **06_DEPENDENCY_GRAPH.md** | 导入/导出关系图。显示文件依赖关系。 | 自动生成 | `npm run gen:graph` |
| **LOGS/** | 历史执行日志。每日会话记录。 | Evaluator | 每个任务后 |

**v2.1 新增**:
- ✨ `06_DEPENDENCY_GRAPH.md` - 自动分析导入/导出关系
- ✨ `03_SERVER_ACTIONS.md` - 现在包含显式的副作用文档

### 2. 📂 _TASK (工作记忆 - 动态执行)

> **原则**：高频变动，任务结束即清理或更新。

| 文件 | 用途 | 创建者 | 使用者 |
|------|------|--------|--------|
| **_PLAN.md** | 战略路线图。"要做什么"和"为什么做"。 | Planner（人/AI） | Planner, Evaluator |
| **_INSTRUCTION.md** | 任务蓝图。"怎么做"及范围约束。 | Planner | Coder |
| **_FIX_INSTRUCTION.md** | 调试模式快速修复。仅限语法/导入错误。 | Evaluator | Coder |

**v2.1 新增**:
- ✨ `_FIX_INSTRUCTION.md` - 紧急修复指令绕过 Planner 处理简单错误

---

## 二、 角色与权限 (Agent Roles & Permissions)

| 角色 | 核心职能 | 读取权限 | 写入权限 | **新能力 (v2.1)** |
| --- | --- | --- | --- | --- |
| **Planner** | 战略转战术 | `_PLAN.md`, 所有 `_DOCS/`, 代码 | `_INSTRUCTION.md` | ✨ 必须检查依赖关系图<br>✨ 可定义参考范围（只读） |
| **Coder** | 最小化执行 | `_INSTRUCTION.md`, `_FIX_INSTRUCTION.md`, 上下文范围, 参考范围 | 仅上下文范围文件 | ✨ 可读取参考范围（只读）<br>✨ 支持调试模式 |
| **Evaluator** | 审计与质量关卡 | `_PLAN.md`, `_INSTRUCTION.md`, 代码差异 | `LOGS/`, `_PLAN.md` 复选框, `_FIX_INSTRUCTION.md` | ✨ 可激活调试模式<br>✨ 验证依赖意识 |
| **Archivist** | 记忆压缩 | `_PLAN.md`, `LOGS/`, 所有 `_DOCS/` | `05_PROJECT_SNAPSHOT.md`, `03_SERVER_ACTIONS.md`, 清理 | ✨ 执行影响审计<br>✨ 记录 API 副作用 |

---

## 三、 标准化作业流程 (SOP)

<img src="https://github.com/JeffSiaYuHeng/dual-brain-system/blob/main/P-C-E-A%20Loop.png?raw=true" alt="P-C-E-A Loop" width="500">

### 🌀 核心循环

系统现在支持**两条执行路径**：

#### **正常流程：P → C → E (→ A)**

```
┌─────────────────────────────────────────────────────────────────┐
│  正常开发循环                                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1️⃣ PLANNER（规划者）：                                         │
│     - 检查 06_DEPENDENCY_GRAPH.md                               │
│     - 定义上下文范围（≤4 个文件，可写）                          │
│     - 定义参考范围（≤2 个文件，只读）                            │
│     - 创建 _INSTRUCTION.md                                       │
│                                                                  │
│  2️⃣ CODER（编码者）：                                          │
│     - 读取 _INSTRUCTION.md                                       │
│     - 读取参考范围获取上下文                                     │
│     - 仅修改上下文范围文件                                       │
│     - 报告完成                                                   │
│                                                                  │
│  3️⃣ EVALUATOR（评估者）：                                      │
│     - 运行构建验证                                               │
│     - 检查范围遵守情况                                           │
│     - 验证依赖意识                                               │
│     - 记录结果 + 勾选复选框                                      │
│     - 如果是简单错误 → 激活调试模式                              │
│                                                                  │
│  决策：还有任务？ → 回到 PLANNER                                 │
│        达到里程碑？ → 运行 ARCHIVIST                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### **调试流程：E → C → E（快速路径）** ⚡ 新增

```
┌─────────────────────────────────────────────────────────────────┐
│  调试模式（绕过 Planner 处理简单错误）                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  EVALUATOR 检测到简单错误：                                      │
│  - 语法错误（缺少分号、括号）                                    │
│  - 缺失导入                                                      │
│  - 拼写错误（变量名不匹配）                                      │
│  - 简单类型注解                                                  │
│                                                                  │
│  ↓                                                               │
│  EVALUATOR 创建 _FIX_INSTRUCTION.md                             │
│  （绕过 Planner - 节省时间）                                     │
│                                                                  │
│  ↓                                                               │
│  CODER 仅应用指定的修复                                          │
│  （不添加功能，不重构）                                          │
│                                                                  │
│  ↓                                                               │
│  EVALUATOR 重新审计：                                            │
│  - 成功 → 继续正常流程                                           │
│  - 仍然失败 → 上报给 Planner                                     │
│  - 最多 2 次迭代，然后上报                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 步骤 1：战术建模（Planner 阶段）

**输入**：`_PLAN.md` 当前焦点
**输出**：`_INSTRUCTION.md`

**流程**（v2.1 更新）：
1. 从 `_PLAN.md` 读取当前焦点
2. **🆕 检查 `06_DEPENDENCY_GRAPH.md`** 了解文件关系
3. 识别目标文件是否为"高影响"（被很多其他文件导入）
4. 定义**上下文范围**（≤4 个文件，可写）
5. **🆕 定义参考范围**（≤2 个文件，只读上下文）
6. 创建包含清晰步骤和约束的 `_INSTRUCTION.md`

**范围决策矩阵**：
| 导入者数量 | 风险 | 行动 |
|-----------|------|------|
| 0-2 | 低 | 标准上下文范围 |
| 3-5 | 中 | 添加警告说明 |
| 6-10 | 高 | 添加 1-2 个导入者到参考范围 |
| 10+ | 关键 | 拆分任务或扩大范围 |

### 步骤 2：隔离手术（Coder 阶段）

**输入**：`_INSTRUCTION.md` 或 `_FIX_INSTRUCTION.md`
**输出**：修改后的代码

**流程**（v2.1 更新）：
1. **🆕 检查存在哪个指令文件**：
   - `_FIX_INSTRUCTION.md` → 调试模式（仅快速修复）
   - `_INSTRUCTION.md` → 正常模式（功能实现）
2. **🆕 读取参考范围**（如果指定）获取使用上下文
3. **仅修改**上下文范围文件
4. **绝不修改**参考范围（只读）
5. 报告完成

**限制**：
- ❌ 不能读取 `_PLAN.md`
- ❌ 不能修改上下文范围外的文件
- ❌ 不能修改参考范围文件（只读）
- ❌ 调试模式下：不能添加功能或重构

### 步骤 3：停机审计（Evaluator 阶段）

**输入**：代码变更、`_INSTRUCTION.md`、`_PLAN.md`
**输出**：日志条目、复选框勾选，或 `_FIX_INSTRUCTION.md`

**流程**（v2.1 更新）：
1. 运行结构同步：`npm run gen:structure`
2. 清除缓存（例如 `rm -rf .next`）
3. **构建验证**：`npm run build`
4. **🆕 如果构建失败且为简单错误**：
   - 语法错误、缺失导入、拼写错误 → **激活调试模式**
   - 创建 `_FIX_INSTRUCTION.md`
   - 绕过 Planner（快速路径）
5. 如果构建通过：
   - 验证范围遵守情况
   - **🆕 检查依赖意识**（如果修改了高影响文件）
   - 创建日志条目
   - 在 `_PLAN.md` 中勾选复选框

**调试模式资格**：
| 错误类型 | 调试模式？ |
|----------|------------|
| 语法错误 | ✅ 是 |
| 缺失导入 | ✅ 是 |
| 拼写错误 | ✅ 是 |
| 逻辑错误 | ❌ 否 → 上报给 Planner |
| 架构违规 | ❌ 否 → 上报给 Planner |

### 步骤 4：熵减（Archivist 阶段 - 触发式）

**触发条件**：里程碑完成、`LOGS/` > 10 个文件，或 `_PLAN.md` > 100 行

**流程**（v2.1 更新）：
1. 扫描 `_PLAN.md` 中已完成的任务
2. 从 `LOGS/` 提取关键决策
3. **🆕 执行影响审计**：
   - 审查已完成任务中的 API 变更
   - **🆕 使用副作用更新 `03_SERVER_ACTIONS.md`**
   - **🆕 如需要添加破坏性变更注释**
   - 记录受影响的文件
4. 更新 `05_PROJECT_SNAPSHOT.md`
5. 清理 `_PLAN.md`（删除已完成任务）
6. 归档或删除旧日志

**副作用文档示例**：
```markdown
### updateUserProfile（在 03_SERVER_ACTIONS.md 中）

**副作用**：
- ⚠️ 更改 `email` 会触发邮件验证工作流
- ⚠️ 更新 `role` 会使缓存的权限失效

**⚠️ 破坏性变更 (2026-02-13)**：
- 返回类型从 `{success: boolean}` 变为 `{success: boolean, userId: string}`
- 使用此操作的前端组件必须更新
```

---

## 四、 新功能 (v2.1)

### 🔗 依赖关系图分析

**目的**：通过理解文件关系防止破坏性变更

**工作原理**：
1. 运行 `npm run gen:graph` 分析导入/导出
2. 生成包含以下内容的 `06_DEPENDENCY_GRAPH.md`：
   - 高影响文件（被很多其他文件导入）
   - 完整依赖映射
   - 反向依赖映射（谁导入了什么）
3. Planner **必须在**定义范围前查阅此文件

**优势**：
- ✅ 100% 准确（基于实际代码）
- ✅ 零 token 成本（预生成）
- ✅ 防止"忘记更新调用者"的错误
- ✅ 实现更智能的参考范围决策

**示例**：
```
任务：修改 lib/utils/validation.ts

Planner 检查 06_DEPENDENCY_GRAPH.md：
→ 发现：validation.ts 被 12 个文件导入
→ 主要导入者：login/page.tsx, UserForm.tsx
→ 决策：将这些添加到参考范围（只读）

结果：Coder 看到 validation 的使用方式，但不修改调用者
```

### 📖 参考范围（只读上下文）

**目的**：提供上下文而不授予修改权限

**工作原理**：
- Planner 在 `_INSTRUCTION.md` 中定义参考范围（≤2 个文件）
- Coder 可以**读取**这些文件以理解使用模式
- Coder **不能修改**参考范围文件

**使用场景**：
- 类型定义文件
- 导入目标文件的文件
- 共享工具以理解参数

**示例**：
```markdown
## 上下文范围（可写）
- lib/utils/date-formatter.ts

## 参考范围（只读）
- app/dashboard/analytics/page.tsx（使用 formatDate）
- components/charts/TimeSeriesChart.tsx（使用 formatTimestamp）
```

### ⚡ 调试模式（快速路径）

**目的**：跳过 Planner 处理简单错误

**工作原理**：
1. Evaluator 检测到简单错误（语法、导入、拼写错误）
2. 直接创建 `_FIX_INSTRUCTION.md`
3. Coder 仅应用修复
4. Evaluator 重新审计
5. 最多 2 次迭代，然后上报给 Planner

**优势**：
- 🚀 简单错误快 3-5 倍
- ⚡ 绕过规划开销
- 🎯 仅聚焦修复

**合格错误**：
- 语法错误（缺少分号、括号）
- 缺失导入
- 变量名拼写错误
- 简单类型注解

**不合格**（上报给 Planner）：
- 逻辑错误
- 架构违规
- 架构问题
- 多个不相关的错误

### 🔍 影响审计（API 完整性）

**目的**：防止前后端契约漂移

**工作原理**：
- Archivist 在里程碑时审查已完成任务
- 识别 API/服务器操作变更
- 使用以下内容更新 `03_SERVER_ACTIONS.md`：
  - 副作用（还会触发什么）
  - 破坏性变更（返回类型变更）
  - 受影响的组件

**优势**：
- ✅ 前端永远不会使用过时的契约
- ✅ 副作用是显式的
- ✅ 破坏性变更有清晰文档

---

## 五、 防御性规则

1. **禁止"顺手修理"**：Coder 发现 Scope 外有错，必须汇报 Planner。严禁私自修改。
2. **禁止"猜测路径"**：若路径找不到，Coder 必须立即 **STOP** 并重新生成结构：`npm run gen:structure`。
3. **单向可见性**：Coder 不知道"大目标"，只执行"原子指令"。防止范围蔓延。
4. **快照即真理**：当 `LOGS` 细节与 `SNAPSHOT` 冲突时，以 `SNAPSHOT` 为准。
5. **🆕 参考范围神圣不可侵犯**：Coder 可以读取但绝不能修改参考范围文件。
6. **🆕 依赖意识必需**：Planner 在定义范围前必须检查依赖关系图。
7. **🆕 调试模式限制**：在上报给 Planner 前最多 2 次迭代。

---

## 六、 按需分配模型 (Cost-aware Routing)

基于任务难度与 Token 消耗的平衡，建议采用以下模型分配方案：

| 角色 | 推荐模型 | 理由 |
|------|---------|------|
| **Planner** | Claude 3.5 Sonnet/4.5, GPT-4o | 需要逻辑推演、依赖分析 |
| **Coder** | Gemini 2.0 Flash, Claude Haiku | 指令明确；追求速度和低成本 |
| **Evaluator** | Claude 3.5 Sonnet/4.5 | 对代码差异和合规性检查敏锐 |
| **Archivist** | Claude 3.5 Sonnet, DeepSeek-V3/R1 | 强大的信息聚合与归纳能力 |

---

## 七、 初始化与快速开始指南

### 前提条件

1. **Node.js**（v14+）用于自动化脚本
2. **新项目仓库**带基本脚手架
3. **访问 AI 模型**符合按需分配路由
4. **30-60 分钟**初始设置时间

### 🚀 项目初始化（执行一次）

#### 步骤 1：安装系统

```bash
# 克隆或复制双脑结构
git clone <your-repo>
cd your-project

# 安装依赖
npm install

# 可选：安装 madge 以获得更好的依赖分析
npm install -g madge
```

#### 步骤 2：创建目录结构

```bash
your-project/
├── _DOCS/                      # 长期记忆
│   ├── 00_STRUCTURE.md         # 自动生成
│   ├── 01_DB_SCHEMA.md         # 手动
│   ├── 02_STYLE_GUIDE.md       # 手动
│   ├── 03_SERVER_ACTIONS.md    # 手动 + Archivist
│   ├── 04_TECH_STACK.md        # 手动
│   ├── 05_PROJECT_SNAPSHOT.md  # Archivist
│   ├── 06_DEPENDENCY_GRAPH.md  # 自动生成 ✨ 新增
│   └── LOGS/
├── _TASK/                      # 工作记忆
│   ├── _PLAN.md
│   ├── _INSTRUCTION.md
│   └── _FIX_INSTRUCTION.md     # ✨ 新增（由 Evaluator 创建）
├── .agent/                     # Agent 技能
│   └── skills/
│       ├── dual-brain-planner/
│       ├── dual-brain-coder/
│       ├── dual-brain-evaluator/
│       └── dual-brain-archivist/
├── scripts/
│   ├── generate-structure.js
│   └── generate-dependency-graph.js  # ✨ 新增
├── src/                        # 你的实际源代码
├── package.json
└── README.md
```

#### 步骤 3：生成文档

```bash
# 生成结构图
npm run gen:structure

# 生成依赖关系图（需要源代码）
npm run gen:graph

# 一次生成两者
npm run gen:all

# 推荐：每次规划会话前运行
npm run pre-plan
```

#### 步骤 4：初始化核心文档

**创建 `_DOCS/01_DB_SCHEMA.md`**（手动）：
```markdown
# 数据库模式

## User 表
| 字段 | 类型 | 约束 |
|------|------|------|
| id | UUID | 主键 |
| email | String | 唯一, 非空 |
| password | String | 已哈希 |
| created_at | Timestamp | 默认: now() |
```

**创建 `_DOCS/02_STYLE_GUIDE.md`**（手动）：
```markdown
# 样式指南

## 颜色
- 主色：#3B82F6
- 错误：#EF4444

## 组件
- 使用 shadcn/ui 组件
- Tailwind 用于样式
```

**创建 `_DOCS/03_SERVER_ACTIONS.md`**（手动）：
```markdown
# 服务器操作

## registerUser
**参数**：`{ email: string, password: string }`
**返回**：`{ success: boolean, userId?: string }`

**副作用**：✨ 新增
- 触发欢迎邮件
- 创建默认用户首选项
```

**创建 `_DOCS/04_TECH_STACK.md`**（手动）：
```markdown
# 技术栈

## 批准的
- Next.js 15
- React 19
- TypeScript
- Prisma ORM
- Tailwind CSS

## 禁止的
- jQuery
- Lodash（使用原生 JS）
```

**创建 `_TASK/_PLAN.md`**：
```markdown
# 战略板

## 路线图
- [ ] 阶段 1：身份验证
- [ ] 阶段 2：仪表板
- [ ] 阶段 3：分析

## 当前焦点
（在这里写下你的第一个任务）

## 备注
（决策、观察、约束）
```

#### 步骤 5：第一次规划会话

```bash
# 1. 生成最新的结构和依赖
npm run pre-plan

# 2. 在 _TASK/_PLAN.md 中更新当前焦点
# 示例："实现用户注册表单"

# 3. 运行 Planner agent（或手动创建 _INSTRUCTION.md）
# Planner 将：
#  - 检查 06_DEPENDENCY_GRAPH.md
#  - 定义上下文范围
#  - 定义参考范围（如需要）
#  - 创建 _INSTRUCTION.md

# 4. 运行 Coder agent 实现

# 5. 运行 Evaluator agent 验证
```

---

### ⚡ 标准工作流

```
┌─────────────────────────────────────────────────────────────┐
│  每次会话前                                                   │
├─────────────────────────────────────────────────────────────┤
│  npm run pre-plan（重新生成结构 + 依赖关系图）               │
└─────────────────────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────────────────────┐
│  1️⃣ PLANNER                                                │
│  - 读取 _PLAN.md 当前焦点                                    │
│  - 检查 06_DEPENDENCY_GRAPH.md  ✨                          │
│  - 定义上下文 + 参考范围                                     │
│  - 创建 _INSTRUCTION.md                                      │
│  时长：5-10 分钟                                             │
└─────────────────────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────────────────────┐
│  2️⃣ CODER                                                  │
│  - 读取 _INSTRUCTION.md（或 _FIX_INSTRUCTION.md）          │
│  - 读取参考范围获取上下文                                    │
│  - 仅修改上下文范围文件                                      │
│  时长：30 分钟 - 2 小时                                      │
└─────────────────────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────────────────────┐
│  3️⃣ EVALUATOR                                              │
│  - 运行构建验证                                              │
│  - 如果是简单错误 → 调试模式  ✨                             │
│  - 如果通过 → 日志 + 勾选复选框                              │
│  时长：10-20 分钟                                            │
└─────────────────────────────────────────────────────────────┘
        ↓
      决策点：
      ├─ 还有任务？ → 回到 PLANNER
      └─ 里程碑？ → 运行 ARCHIVIST
```

---

### 📋 具体示例：构建你的第一个功能

**场景**：构建用户身份验证系统

#### **迭代 1：注册表单**

```
🧠 PLANNER：
   npm run pre-plan

   检查 06_DEPENDENCY_GRAPH.md：
   - 还没有依赖（新文件）

   创建 _INSTRUCTION.md：

   ## 上下文范围
   - src/components/auth/RegisterForm.tsx（新）
   - src/actions/user.ts（新）

   ## 参考范围
   （无 - 新功能）

   ## 步骤
   1. 创建带邮箱/密码字段的 RegisterForm
   2. 添加验证（Zod schema）
   3. 创建 registerUser 服务器操作
   4. 使用 bcrypt 哈希密码

   ## 超出范围
   - 登录表单（下一个任务）
   - 邮箱验证（稍后）

🔧 CODER：
   读取 _INSTRUCTION.md
   ✅ 创建 RegisterForm.tsx
   ✅ 创建带 registerUser 操作的 user.ts
   ✅ 本地测试
   报告："注册表单完成"

🔍 EVALUATOR：
   npm run gen:structure（用新文件更新）
   npm run build → ❌ 失败

   错误："缺少导入：bcrypt"

   🆕 调试模式激活：
   创建 _FIX_INSTRUCTION.md：

   ## 错误类型
   IMPORT_MISSING

   ## 目标文件
   - src/actions/user.ts:3

   ## 需要的修复
   添加：import bcrypt from 'bcrypt'

   移交给 Coder

🔧 CODER（调试模式）：
   读取 _FIX_INSTRUCTION.md
   ✅ 添加缺少的导入
   报告："导入已添加"

🔍 EVALUATOR（重新审计）：
   npm run build → ✅ 成功
   ✅ 创建日志条目
   ✅ 勾选复选框：[x] 注册表单

   决策：更多身份验证任务 → 回到 PLANNER
```

#### **迭代 2：登录表单**

```
🧠 PLANNER：
   npm run pre-plan

   🆕 检查 06_DEPENDENCY_GRAPH.md：
   - user.ts 被 RegisterForm.tsx 导入
   - 风险：低（只有 1 个导入者）

   创建 _INSTRUCTION.md：

   ## 上下文范围
   - src/components/auth/LoginForm.tsx（新）
   - src/actions/user.ts（修改 - 添加 loginUser）

   ## 参考范围
   - src/components/auth/RegisterForm.tsx
     （查看 registerUser 如何调用）

   ## 步骤
   1. 创建类似 RegisterForm 的 LoginForm
   2. 向 user.ts 添加 loginUser 操作
   3. 使用 bcrypt.compare 验证密码

🔧 CODER：
   读取 _INSTRUCTION.md
   🆕 读取参考范围：RegisterForm.tsx（仅上下文）
   ✅ 创建 LoginForm（匹配 RegisterForm 模式）
   ✅ 向 user.ts 添加 loginUser
   报告："登录表单完成"

🔍 EVALUATOR：
   npm run build → ✅ 成功
   ✅ 验证范围：仅修改了 LoginForm 和 user.ts
   ✅ 验证：未修改 RegisterForm（参考范围）
   ✅ 创建日志条目
   ✅ 勾选复选框：[x] 登录表单

   决策：身份验证功能完成 → 运行 ARCHIVIST
```

#### **Archivist 阶段**

```
📚 ARCHIVIST：
   扫描 _PLAN.md：
   - [x] 注册表单
   - [x] 登录表单

   审查 LOGS/：
   - 2026-02-13.md：注册 + 调试导入修复
   - 2026-02-14.md：登录完成

   🆕 影响审计：
   检查 03_SERVER_ACTIONS.md：

   更新为：

   ### registerUser
   **参数**：{ email, password }
   **返回**：{ success, userId }
   **副作用**：
   - 触发欢迎邮件工作流
   - 创建用户会话

   ### loginUser
   **参数**：{ email, password }
   **返回**：{ success, token }
   **副作用**：
   - 使之前的会话失效
   - 记录登录尝试

   更新 05_PROJECT_SNAPSHOT.md：

   ## 已完成功能
   - [x] 用户身份验证
     - 带验证的注册
     - 带会话管理的登录
     - 密码哈希（bcrypt）

   清理：
   ✅ 从 _PLAN.md 删除已完成任务
   ✅ 归档旧日志
   ✅ 重置 _INSTRUCTION.md

   准备好进行：阶段 2 - 仪表板
```

---

### 🛠️ 快速参考：文件和脚本

| 命令 | 目的 | 何时运行 |
|------|------|----------|
| `npm run gen:structure` | 生成文件树 | 规划前、添加文件后 |
| `npm run gen:graph` | 生成依赖映射 | 规划前、更改导入后 |
| `npm run gen:all` | 生成两者 | ✅ 每次会话前推荐 |
| `npm run pre-plan` | gen:all 的别名 | 使用这个！ |

| 文件 | 谁读取它 | 谁写入它 |
|------|---------|---------|
| `06_DEPENDENCY_GRAPH.md` | Planner | 自动生成 |
| `_INSTRUCTION.md` | Coder | Planner |
| `_FIX_INSTRUCTION.md` | Coder | Evaluator（调试模式） |
| 参考范围文件 | Coder（只读） | - |

---

### ⚠️ 常见陷阱和解决方案

| 陷阱 | 症状 | 解决方案 (v2.1) |
| --- | --- | --- |
| **破坏性变更** | 修改的文件破坏了调用者 | 首先检查 `06_DEPENDENCY_GRAPH.md` ✨ |
| **范围蔓延** | Coder 修改了太多文件 | 上下文范围限制在 4 个文件 |
| **简单错误循环** | Planner 浪费时间处理拼写错误 | 使用调试模式（E→C→E）✨ |
| **参考范围违规** | Coder 修改只读文件 | 在 Coder 技能中强制只读 ✨ |
| **API 漂移** | 前端使用过时的契约 | 运行影响审计（Archivist）✨ |
| **虚构路径** | Coder 尝试不存在的文件 | 运行 `npm run gen:structure` |
| **失去上下文** | Coder 读取 _PLAN.md | 严格执行仅指令读取 |
| **过时的依赖** | 未检测到破坏性变更 | 定期运行 `npm run gen:graph` |

---

### 📚 其他资源

- **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** - 单页速查表
- **[SKILLS_UPDATE_SUMMARY.md](SKILLS_UPDATE_SUMMARY.md)** - 完整 v2.1 变更日志
- **[DEPENDENCY_GRAPH_SETUP.md](DEPENDENCY_GRAPH_SETUP.md)** - 依赖分析设置指南
- **[UPDATE_VERIFICATION.md](UPDATE_VERIFICATION.md)** - v2.1 中的变更内容

---

### 🎯 成功指标

使用 v2.1，你应该看到：
- ✅ 更少的破坏性变更（依赖意识）
- ✅ 更快的简单错误修复（调试模式）
- ✅ 更好的实现上下文（参考范围）
- ✅ 更清晰的 API 契约（影响审计）
- ✅ 降低的 token 成本（预生成分析）

---

**版本**：2.1
**最后更新**：2026-02-13
**状态**：生产就绪

如有疑问、问题或贡献，请参见 [GitHub 仓库](https://github.com/JeffSiaYuHeng/dual-brain-system)。

---

*使用双脑系统愉快地构建！🚀*
