# 双脑系统 (Dual-Brain System) 规范文档 (V2.0)💡

*为了优化 Side Project 的产出比，设计的一套基于元认知理论的 AI 协作协议 🤖*


# Introduction

关于这个理念的落地，我结合了之前开会时Hiredly InnoTech团队CTO和我提及的“规格驱动开发（Spec-Driven Development）”概念，并参考了之前读过的丹尼尔·卡尼曼在《思考，快与慢》中提出的系统1与系统2理论，创建了一个基于Markdown的文件系统。书中定义的系统1是快速、直觉且感性的，负责日常的自动化决策；而系统2则是缓慢、审慎且逻辑化的，负责复杂的分析性推理。我设计的这套系统是在模拟这种认知运作方式，将“规格”作为引导AI**系统2深度思考的媒介**并把任务细分为能够为**系统1的直觉快速处理常规任务**，更能通过规格文档的约束进入系统2的慢思考模式，对复杂的业务逻辑进行严密的推演和架构设计。

在目前的系统实践中，我设计了由**长期记忆**与**短期记忆**组成的规格结构。长期记忆涵盖了项目结构、数据库关系文档、软件需求说明（SRS）以及服务端动作逻辑，构成了项目的核心知识底座；短期记忆则通过_Plan.md文件来定义具体任务的行动方案，明确目标、范围及验证清单，从而形成任务的初步全局视图。随后，系统会基于计划生成_Instruction.md，将构思转化为一系列可执行的指令流。这种将元认知思维引入项目结构的做法，结合近期推出的“Agent-skill（代理技能）”机制，使AI能够脱离固定的职业标签，转而以技能和规格为驱动，实现更具灵活性和自主性的深度协作。

其实更主要的是为了节省开发Side Project的成本，能够透过不同的IDE或者CLI来协作达到低成本且通用于不同模型的Adapter的感觉，他们只需要明白目前的任务以及结构作为输入的媒介，这就很大程度的减少了Token的消耗，并且规范了它们能够触及的范围，能够更加全面的掌控整体的进程，但是比较小型的Task就还是一样Single File， Single Prompt就能够解决了，不需要到杀鸡用牛刀的感觉，所以整个模式在运作的时候其实也是很多人工的介入，之后还是会继续优化, 到目前为止整体都是靠直觉驱动来设计的，实用性有待考证。

---

## 一、 存储架构 (Memory Structure)

系统分为 **长期记忆 (_DOCS)** 和 **工作记忆 (_TASK)**，实现“事实”与“意图”的物理隔离。

### 1. 📂 _DOCS (长期记忆 - 静态真理)

> **原则**：AI 只读不写（除 Evaluator 记录日志与 Archivist 更新快照外）。
> 
- **00_STRUCTURE.md**：文件地图。防止 AI 虚构路径。
- **01_DB_SCHEMA.md**：后端真理。字段、类型、权限的唯一来源。
- **02_STYLE_GUIDE.md**：视觉与交互规范。确保 UI 一致性。
- **03_SERVER_ACTIONS.md**：API 协议。定义前后端通讯黑盒。
- 04_TECH_STACK.md : 定义项目中**被官方允许使用的技术栈**。
- 05_PROJECT_SNAPSHOT.md: 项目高密度快照。由 Archivist 维护，记录当前功能状态与技术决策结论。
- **LOGS/**：历史快照。记录每一个 Loop 的技术实现细节。
    - LOG(format).md: 提供Log的format.

### 2. 📂 _TASK (工作记忆 - 动态执行)

> **原则**：高频变动，任务结束即清理或更新。
> 
- **_INSTRUCTION.md**：**任务卡 (AI Blueprint)**。记录“怎么做”和“限制在哪”。

---

## 二、 角色与权限 (Agent Roles)

- **_PLAN.md**：**战略板 (Human Focus)**。记录“要做什么”和“为什么做”。

**Folder Strucuture**

**Roles**

| 角色 | 核心职能 | 准入权限 (Read) | 写入权限 (Write) |
| --- | --- | --- | --- |
| **Planner** | 战略转战术 | _PLAN, _DOCS, 源代码 | _INSTRUCTION |
| **Coder** | 最小化执行 | _INSTRUCTION, Scope 内文件 | Scope 内文件 |
| **Evaluator** | 审计与闭环 | _PLAN, _INSTRUCTION, diff | LOGS, _PLAN (勾选) |
| Archivist | 熵减与压缩 | _PLAN, LOGS, _DOCS | _SNAPSHOT, _PLAN (清理) |

---

## 三、 标准化作业流程 (SOP)

### 🌀 核心循环：P-C-E-A Loop

### Step 1: 战术建模 (Planner 阶段)

- **动作**：根据 _PLAN.md 的 CURRENT FOCUS 和 _SNAPSHOT.md 的当前状态，输出 _INSTRUCTION.md。
- **指标**：定义 **Context Scope** (≤ 4个文件) 和 **Out of Scope**。

### Step 2: 隔离手术 (Coder 阶段)

- **动作**：**盲目**执行指令。禁止读取 _PLAN.md，禁止修改 Scope 外文件。

### Step 3: 停机审计 (Evaluator 阶段)

- **动作**：核对代码是否实现 _PLAN 意图，检查是否有 Scope 溢出，并在 LOGS/ 生成记录。

### Step 4: 熵减压缩 (Archivist 阶段 - 触发式)

- **触发**：当里程碑完成、或 LOGS/ 文件过多、或 _PLAN.md 已办事项堆积时。
- **动作**：
    1. 提取 LOGS/ 中的关键决策进入 04_PROJECT_SNAPSHOT.md。
    2. 清理 _PLAN.md 中已完成的任务。
    3. 重置 _INSTRUCTION.md 为待机状态。

---

## 四、 防御性规则 (Defensive Rules)

1. **禁止“顺手修理”**：Coder 发现 Scope 外有错，必须汇报 Planner，严禁私自修改。
2. **禁止“猜测路径”**：若结构文档未更新导致路径找不到，Coder 必须立即 **Stop**。
3. **单向可见性**：Coder 不知道“大目标”，只执行“原子指令”。防止 AI 为了凑合大目标而引入系统性 Bug。
4. **快照即真理**：当 LOGS 细节与 SNAPSHOT 冲突时，以 SNAPSHOT 为准。

## 五、 按需分配模型 (Cost-aware Routing)

基于任务难度与 Token 消耗的平衡，建议采用以下模型分配方案：

- **Planner (高级大脑)**：使用 **Claude 4.5 Sonnet** 或 **GPT-4o**。需要极强的逻辑推演能力。
- **Coder (手术刀)**：使用 **Gemini 2.5 Flash**。指令已明确，追求速度与低廉的上下文成本。
- **Evaluator (审计员)**：使用 **Claude 4.5 Sonnet**。对代码 Diff 和合规性检查非常敏锐。
- **Archivist (记忆官)**：使用 **Claude 4.5 Sonnet** 或 **DeepSeek-V3/R1**。需要极强的信息聚合与归纳能力，将冗长的日志压缩为高密度结论。

---
## 六、 初始化与快速开始指南

### 前置条件

开始前，确保你拥有：

1. **一个新的项目仓库**，包含基础脚手架（Next.js、Django、Flutter 等）
2. **AI 模型的访问权限**，与第五章的成本感知路由相符
3. **基本的技术栈知识**
4. **30-60 分钟**的初始化时间
5. **Markdown 编辑器**（VS Code、Obsidian 等）

### 🚀 项目初始化清单（仅需执行一次）

#### 第一步：创建目录结构

```bash
your-project/
├── _DOCS/                 # 长期记忆
│   ├── 00_STRUCTURE.md
│   ├── 01_DB_SCHEMA.md
│   ├── 02_STYLE_GUIDE.md
│   ├── 03_SERVER_ACTIONS.md
│   ├── 04_TECH_STACK.md
│   ├── 05_PROJECT_SNAPSHOT.md
│   └── LOGS/
│       └── LOG(format).md
├── _TASK/                 # 工作记忆
│   ├── _PLAN.md
│   ├── _INSTRUCTION.md
│   └── _RESULT.md (可选：用于评估笔记)
├── src/                   # 你的实际源码
├── README.md
└── .gitignore
```

#### 第二步：初始化长期记忆 (_DOCS)

**创建 `_DOCS/00_STRUCTURE.md`：**
- 映射实际代码库的**整体目录结构**
- 列出所有关键文件夹及其用途
- 标注使用的技术/框架
- 例：[参考工作区的结构示例]

**创建 `_DOCS/01_DB_SCHEMA.md`：**
- 定义你的数据库模式（表、字段、类型、关系）
- 记录任何约束或索引
- 包含迁移信息（如适用）
- 这是**后端数据的唯一真理来源**

**创建 `_DOCS/02_STYLE_GUIDE.md`：**
- 文档 UI/UX 约定（颜色、排版、间距）
- 定义交互模式（弹窗、表单、导航）
- 包含组件命名约定
- 链接到设计系统（如有）

**创建 `_DOCS/03_SERVER_ACTIONS.md`：**
- 列出所有 API 端点或服务器操作，包含：
  - HTTP 方法 / 函数名称
  - 请求/响应模式
  - 身份验证需求
  - 错误代码
- 格式为参考表或 OpenAPI 规范

**创建 `_DOCS/04_TECH_STACK.md`：**
- 列出批准的框架、库和工具
- 定义版本约束
- 标注任何已弃用或禁用的库
- 包含开发环境设置要求

**创建 `_DOCS/05_PROJECT_SNAPSHOT.md`：**
- 高级功能完成状态
- 按功能分解（**不是**逐行代码）
- 已知问题和技术债
- 关键指标（组件数、测试覆盖率等）
- 这由 **Archivist 角色维护**

**创建 `_DOCS/LOGS/LOG(format).md`：**
- 开发会话日志的模板
- 包含：目标、已完成任务、问题、下一阶段重点
- 命名约定：`YYYY-MM-DD.md` 或 `YYYY-MM-DD_任务名.md`

#### 第三步：初始化工作记忆 (_TASK)

**创建 `_TASK/_PLAN.md`：**

```markdown
# 战略板 (计划)

## 路线图
1. [ ] Phase 1: 核心功能 A
2. [ ] Phase 2: 核心功能 B
3. [ ] Phase 3: 集成与优化

## CURRENT FOCUS
(用自然语言写出你的第一个任务)
- 例：用 Email/密码实现用户认证流程

## 笔记/便签
- 任何观察、决策或约束条件
```

**创建 `_TASK/_INSTRUCTION.md`：**

保持**空白或待机状态**，直至 Planner 阶段填充。

```markdown
# 任务指令

## 状态
**等待 Planner 分配**

(当从 _PLAN.md 选择 CURRENT FOCUS 时，此文件将被填充)
```

#### 第四步：提交到 Git（可选但推荐）

```bash
git add _DOCS/ _TASK/
git commit -m "init: 初始化双脑系统结构"
```

---

### ⚡ 标准工作流程：P-C-E 循环（迭代执行直到组件完成）

> **关键概念**：Planner → Coder → Evaluator 循环**对每一个原子任务重复**，直到完整组件/功能完成。**之后才执行 Archivist**。

```
┌─────────────────────────────────────────────────────────────────┐
│  一个工作循环（重复直到组件完成）                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1️⃣ PLANNER: 定义任务 → _INSTRUCTION.md                       │
│  2️⃣ CODER: 执行指令                                            │
│  3️⃣ EVALUATOR: 审查与记录结果                                  │
│                                                                  │
│  决策点：                                                        │
│  ├─ 组件完成了吗? → 执行 ARCHIVIST                            │
│  └─ 还有更多任务? → 重复 (返回 PLANNER 选择下一个任务)        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────────────────────────┐
│  ARCHIVIST（运行一次，在组件完成时）                           │
│  - 合并日志 → 更新 _SNAPSHOT.md                               │
│  - 清理 _PLAN.md                                              │
│  - 归档旧日志                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

#### **阶段 1：Planner** 🧠

**何时**：每个开发任务之前
**谁**：你（人工操作者）或一个战略性 AI 代理
**耗时**：5-10 分钟

**做什么**：

1. **阅读** `_PLAN.md` 并确认下一个 **CURRENT FOCUS** 任务
2. **检查** `_SNAPSHOT.md` 以了解已完成的工作和阻碍因素
3. **审视** 相关的 `_DOCS/` 文件（结构、模式、API 规范、样式指南）
4. **生成** `_INSTRUCTION.md` 包含：
   - 清晰的**上下文**（为什么这个任务很重要）
   - 明确的**上下文范围**（≤4 个文件要修改）
   - 编号的、祈使式的**步骤**（要做什么，而非怎么做）
   - **超出范围**部分（不能修改什么）
   - **约束与规则**（模式、工具、需求）
   - **质量清单**（如何验证成功）

**示例输出** (_INSTRUCTION.md)：
```markdown
# 任务指令：实现用户注册表单

## 上下文
让新用户能创建账户。解锁完整的认证流程。
来自：_PLAN.md - Phase 1: 认证

## 上下文范围（严格）
- `src/components/Auth/RegisterForm.tsx`
- `src/actions/user.ts`
- `prisma/schema.prisma` (仅模式更新)

## 执行步骤
1. 扩展 Prisma User 模型，添加 `emailVerified` 和 `verificationCode` 字段。
2. 创建 RegisterForm 组件，实现 email/密码验证。
3. 添加 `registerUser` 服务器操作，实现密码哈希和 DB 存储。
4. 向前端返回成功/错误消息。

## 超出范围
- 暂不实现邮件验证。
- 暂不创建登录页面。
- 暂不修改认证中间件。

## 约束与规则
- 使用 bcrypt 进行密码哈希（已在 `04_TECH_STACK.md` 中）。
- 在 DB 执行前验证邮箱格式。
- 生产代码中禁止 console.logs。

## 质量清单
- [ ] 表单验证对无效邮箱工作。
- [ ] 密码在存储前被哈希。
- [ ] 错误消息对用户友好。
- [ ] 代码遵循现有代码库的模式。
```

---

#### **阶段 2：Coder** 🔧

**何时**：`_INSTRUCTION.md` 准备就绪后
**谁**：实现 AI 代理或你（开发者）
**耗时**：30 分钟 - 2 小时（取决于任务范围）

**做什么**：

1. **仅读** `_INSTRUCTION.md`（**不要**读 `_PLAN.md` 或 `_SNAPSHOT.md`）
2. **检查** `_DOCS/` 中的参考文件以获取上下文（模式、样式指南、API 规范）
3. **按顺序执行**各步骤，**仅修改上下文范围内的文件**
4. **局部测试**确保所有步骤按预期工作
5. **立即停止**如果需要修改超出范围的文件（向 Planner 报告）
6. **反馈总结**包含更改摘要和任何阻碍

**关键规则**：
- ❌ **不要**读 `_PLAN.md`（防止范围蔓延和偏见）
- ❌ **不要**修改上下文范围外的文件
- ❌ **不要**在指令之外进行"改进"或"优化"
- ✅ 如果指令模糊，要求澄清
- ✅ 根据质量清单进行测试后再报告完成

---

#### **阶段 3：Evaluator** 🔍

**何时**：Coder 报告完成后
**谁**：你、另一个开发者或审计 AI 代理
**耗时**：10-20 分钟

**做什么**：

1. **审查**代码更改（运行 `git diff` 查看所有修改）
2. **验证** `_INSTRUCTION.md` 是否被完全执行
3. **检查**无文件被修改超出上下文范围
4. **运行**代码：
   - 执行功能相关的测试
   - 手动测试（快乐路径 + 边界情况）
   - 检查与现有代码的集成问题
5. **记录发现**到 `_DOCS/LOGS/YYYY-MM-DD.md`：
   - 目标与已完成任务 ✅
   - 发现的问题或 Bug ⚠️
   - 下一步工作 🎯
   - 任何技术决策

**日志示例** (_DOCS/LOGS/2026-02-10.md)：
```markdown
# 日志：2026-02-10

## 目标
- 实现用户注册表单和验证。

## 已完成的任务
- [x] 扩展 Prisma User 模型，添加 emailVerified 字段。
- [x] 创建 RegisterForm.tsx，实现 email/密码验证。
- [x] 实现 `registerUser` 服务器操作，使用 bcrypt 哈希。
- [x] 手动测试通过（快乐路径和验证错误）。

## 问题与阻碍
- 邮件验证 OTP 尚未集成（意图之外）。
- 注意到表单在移动设备上需要填充（轻微 UX 问题，无阻碍）。

## 下一个任务重点
- 实现登录表单（结构类似注册）。
- 然后：添加邮件验证流程。

## 技术决策
- 使用 bcrypt、saltRounds=10（遵循安全最佳实践）。
```

---

#### **评估后的决策** 🔄

Evaluator 完成后：

- **✅ 如果功能已完成**：跳到**Archivist 阶段**（见下文）
- **❌ 如果还需要更多任务**：回到**阶段 1 (Planner)**，选择下一个 CURRENT FOCUS

**例**：
```
Evaluator: "注册表单完成了。准备好做登录了吗？"
Planner: "是的。现在重点：实现用户登录表单。"
└─ 生成新的 _INSTRUCTION.md → Coder 执行 → Evaluator 审查 → 重复直到认证功能完成
```

---

#### **阶段 4：Archivist** 📚

**何时**：**仅当一个完整的组件/功能完成时**（在多次 P-C-E 循环后）
**谁**：你或 AI Archivist
**耗时**：15-30 分钟
**频率**：每个功能完成一次（例：认证后、购物车后、订单后等）

**触发示例**：
- ✅ "认证功能完成"（注册 + 登录 + 验证都完成了）
- ✅ "购物车功能完成"（添加/删除 + 折扣 + 结账都完成了）
- ❌ 不是每个单一任务后（开销过大）

**做什么**：

1. **压缩知识**：从 `LOGS/` 提取关键决策并合并到 `05_PROJECT_SNAPSHOT.md`
2. **更新路线图**：在 `_PLAN.md` 中标记已完成项目并清理旧条目
3. **重置指令**：清空 `_INSTRUCTION.md` 回到待机状态
4. **巩固文档**：将任何新发现移入 `_DOCS/`（模式变化、新模式等）
5. **归档日志**：将旧的日常日志移到历史文件夹或删除（如已合并）

**示例**（Archivist 执行前）：
```
_PLAN.md:
- [x] Phase 1.1: 用户注册 (完成)
- [x] Phase 1.2: 用户登录 (完成)
- [x] Phase 1.3: 邮件验证 (完成)
- [ ] Phase 2: 购物车

LOGS/:
- 2026-02-07.md: 注册完成
- 2026-02-08.md: 登录完成
- 2026-02-09.md: 发现邮箱验证 Bug
- 2026-02-10.md: Bug 修复，验证完成
```

**Archivist 执行后**：
```
_PLAN.md:
- [ ] Phase 2: 购物车

_SNAPSHOT.md (更新为):
- [x] Phase 1: 用户认证 - 完成 ✅
  - 注册：email/密码注册，包含验证
  - 登录：基于会话的认证，安全令牌
  - 邮件验证：基于 OTP 的确认
  - 已知问题：需要 OTP 请求的速率限制（Phase 3 待办）
  - 测试覆盖率：认证流程 87%

LOGS/:
- LOG(format).md (模板仅)
- (旧的日常日志已删除或归档)
```

---

### 📋 具体例：构建你的第一个功能

假设你在构建一个**食品订购系统**，从**餐厅列表**开始。

---

**迭代 1：组件设置**

```
🧠 PLANNER 阶段：
   更新 _PLAN.md CURRENT FOCUS:
   "构建餐厅列表页面 - 显示所有餐厅，包含名称、评分、菜系"
   
   创建 _INSTRUCTION.md，范围为：
   - src/pages/restaurants.tsx
   - src/actions/restaurant.ts
   - src/components/RestaurantCard.tsx

🔧 CODER 阶段：
   ✅ 创建 RestaurantCard 组件
   ✅ 实现 getRestaurants() 操作
   ✅ 构建页面布局，使用网格
   ✅ 本地测试通过

🔍 EVALUATOR 阶段：
   ✅ 代码审查通过
   ✅ 手动测试 OK
   ✅ 日志："组件渲染完成。准备下一步搜索/过滤。"

❓ 决策：还有更多餐厅功能工作? 是 → 继续循环
```

**迭代 2：搜索与过滤**

```
🧠 PLANNER 阶段：
   更新 CURRENT FOCUS: "为餐厅列表添加搜索和菜系过滤"
   
   创建 _INSTRUCTION.md，范围为（不同任务，相同文件）：
   - src/components/RestaurantFilter.tsx (新)
   - src/actions/restaurant.ts (修改)
   - src/pages/restaurants.tsx (修改)

🔧 CODER 阶段：
   ✅ 创建 RestaurantFilter 组件
   ✅ 为 getRestaurants() 添加过滤逻辑
   ✅ 更新页面使用过滤器状态
   ✅ 本地测试通过

🔍 EVALUATOR 阶段：
   ✅ 代码审查通过
   ✅ 手动测试 OK
   ✅ 日志："搜索和过滤工作正常。餐厅功能完成。"

❓ 决策：功能完成? 是 → 运行 ARCHIVIST
```

**ARCHIVIST 阶段：合并与继续**

```
📚 ARCHIVIST:
   ✅ 更新 _SNAPSHOT.md:
      [x] 餐厅列表 - 完成
      - 显示所有餐厅
      - 按名称搜索
      - 按菜系过滤
      
   ✅ 更新 _PLAN.md:
      删除已完成的餐厅任务
      标记 "Phase 1: 餐厅列表" 为完成
      
   ✅ 归档 LOGS:
      移除 2026-02-07.md 和 2026-02-08.md（旧日志）
      
   ✅ 清理 _INSTRUCTION.md → 待机状态

😊 准备好下一功能："Phase 2: 购物车"
```

---

### 🛠️ 快速参考：何时使用每个文件

| 问题 | 答案位置 | 谁读 |
| --- | --- | --- |
| "登录 API 在哪？" | `03_SERVER_ACTIONS.md` | Planner, Coder |
| "User 表有哪些字段？" | `01_DB_SCHEMA.md` | 所有人 |
| "我能用 Vue.js 吗？" | `04_TECH_STACK.md` | Planner |
| "哪些功能已完成？" | `05_PROJECT_SNAPSHOT.md` | Planner, Archivist |
| "上一个会话发生了什么？" | `LOGS/YYYY-MM-DD.md` | Evaluator, Archivist |
| "我下一步应该构建什么？" | `_PLAN.md` (CURRENT FOCUS) | Planner |
| "我确切地应该怎样构建它？" | `_INSTRUCTION.md` | Coder |

---

### ⚠️ 常见陷阱与解决方案

| 陷阱 | 症状 | 修复 |
| --- | --- | --- |
| **范围蔓延** | _INSTRUCTION.md 太大或模糊 | 限制上下文范围为 4 个文件；使用祈使式语言 |
| **虚构路径** | Coder 试图编辑不存在的文件 | 在 00_STRUCTURE.md 中首先列出所有路径 |
| **丢失上下文** | Coder 读 _PLAN.md 并偏离 | 严格执行"Coder 仅读 _INSTRUCTION.md"规则 |
| **陈旧快照** | LOGS 和 _SNAPSHOT.md 之间冲突 | 定期运行 Archivist 阶段（至少每周）|
| **Token 膨胀** | 又长又无焦点的指令 | 将指令修剪为原子的、单个功能的任务 |
| **无文档** | 难以引入另一个 AI/开发者 | 结构性更改后**立即**更新 _DOCS/ |

---