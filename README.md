# Dual-Brain System (双脑系统) Specification Document (V2.0) 💡

📖 **[中文版本](README(CN).md)** | English Version

*An AI collaboration protocol based on metacognition theory, designed to optimize the output-to-input ratio of side projects 🤖*

---

# Introduction

Regarding the implementation of this concept, I combined the "Spec-Driven Development" (SDD) concept mentioned to me by the CTO of the Hiredly InnoTech team during a previous meeting, and referenced Daniel Kahneman's "System 1 and System 2" theory from "Thinking, Fast and Slow" to create a Markdown-based file system. System 1, as defined in the book, is fast, intuitive, and emotional, responsible for daily automated decisions; while System 2 is slow, deliberate, and logical, responsible for complex analytical reasoning. The system I designed simulates this cognitive operation by using "specifications" as a medium to guide AI into **System 2 deep thinking**, while breaking down tasks into routine actions that can be quickly processed by **System 1 intuition**. Through the constraints of specification documents, the AI is encouraged to enter the System 2 slow thinking mode for rigorous deduction and architectural design of complex business logic.

In current system practice, I've designed a specification structure consisting of **Long-term Memory** and **Short-term Memory**. Long-term memory covers project structure, database schema, software requirement specifications (SRS), and server-side action logic, forming the core knowledge base of the project. Short-term memory uses the `_PLAN.md` file to define specific task action plans, clarifying goals, scope, and verification checklists, thus forming an initial global view of the task. Subsequently, the system generates `_INSTRUCTION.md` based on the plan, transforming concepts into a series of executable instruction streams. This approach of introducing metacognitive thinking into project structures, combined with the recently launched "Agent-skill" mechanism, allows AI to move beyond fixed professional labels and instead operate as a skill- and spec-driven entity, achieving more flexible and autonomous deep collaboration.

Actually, the main goal is to save the cost of developing Side Projects. It feels like a low-cost, universal Adapter for different models through collaboration across various IDEs or CLIs. They only need to understand the current task and structure as input media, which significantly reduces Token consumption and regulates their reach, allowing for more comprehensive control over the overall process. However, for smaller tasks, a Single File and Single Prompt approach still works fine; no need to use a sledgehammer to crack a nut. Therefore, the entire mode involves a lot of manual intervention during operation and will continue to be optimized. As of now, the overall design is driven by intuition, and its practicality remains to be verified.

---

## I. Memory Structure

The system is divided into **Long-term Memory (_DOCS)** and **Working Memory (_TASK)**, achieving physical isolation between "facts" and "intentions".

### 1. 📂 _DOCS (Long-term Memory - Static Truth)

> **Principle**: AI is Read-only (except for Evaluator logging and Archivist snapshot updates).

- **00_STRUCTURE.md**: File map. Prevents AI from hallucinating paths.
- **01_DB_SCHEMA.md**: Backend truth. The sole source for fields, types, and permissions.
- **02_STYLE_GUIDE.md**: Visual and interaction specifications. Ensures UI consistency.
- **03_SERVER_ACTIONS.md**: API protocol. Defines the frontend-backend communication black box.
- **04_TECH_STACK.md**: Defines the **officially permitted tech stack** in the project.
- **05_PROJECT_SNAPSHOT.md**: High-density project snapshot. Maintained by the Archivist, recording current feature status and technical decision conclusions.
- **LOGS/**: Historical snapshots. Records technical implementation details for every loop.
    - **LOG(format).md**: Provides the log format.

### 2. 📂 _TASK (Working Memory - Dynamic Execution)

> **Principle**: High-frequency changes, cleared or updated upon task completion.

- **_INSTRUCTION.md**: **Task Card (AI Blueprint)**. Records "how to do it" and "constraints".
- **_PLAN.md**: **Strategy Board (Human Focus)**. Records "what to do" and "why it's done".

---

## II. Agent Roles & Permissions

| Role | Core Responsibility | Access Permissions (Read) | Write Permissions (Write) |
| --- | --- | --- | --- |
| **Planner** | Strategy to Tactics | _PLAN, _DOCS, Source Code | _INSTRUCTION |
| **Coder** | Minimal Execution | _INSTRUCTION, Files within Scope | Files within Scope |
| **Evaluator** | Audit & Closing Loop | _PLAN, _INSTRUCTION, Diff | LOGS, _PLAN (Checked) |
| **Archivist** | Entropy Reduction & Compression | _PLAN, LOGS, _DOCS | _SNAPSHOT, _PLAN (Cleanup) |

---

## III. Standard Operating Procedure (SOP)
<img src="https://github.com/JeffSiaYuHeng/dual-brain-system/blob/main/P-C-E-A%20Loop.png?raw=true" alt="P-C-E-A Loop" width="500">

### 🌀 Core Cycle: P-C-E-A Loop

### Step 1: Tactical Modeling (Planner Phase)

- **Action**: Outputs `_INSTRUCTION.md` based on the `CURRENT FOCUS` in `_PLAN.md` and the current state in `_SNAPSHOT.md`.
- **Metrics**: Defines **Context Scope** (≤ 4 files) and **Out of Scope**.

### Step 2: Isolated Surgery (Coder Phase)

- **Action**: **Blindly** executes instructions. Prohibited from reading `_PLAN.md` or modifying files outside of Scope.

### Step 3: Shutdown Audit (Evaluator Phase)

- **Action**: Verifies if the code implements the `_PLAN` intention, checks for Scope overflow, and generates records in `LOGS/`.

### Step 4: Entropy Reduction & Compression (Archivist Phase - Trigger-based)

- **Trigger**: Upon milestone completion, excessive `LOGS/` files, or an accumulation of completed items in `_PLAN.md`.
- **Action**:
    1. Extracts key decisions from `LOGS/` into `05_PROJECT_SNAPSHOT.md`.
    2. Clears completed tasks in `_PLAN.md`.
    3. Resets `_INSTRUCTION.md` to standby state.

---

## IV. Defensive Rules (Defensive Rules)

1. **No "Convenience Fixes"**: If the Coder finds an error outside of Scope, they must report to the Planner. Unauthorized modification is strictly forbidden.
2. **No "Path Guessing"**: If a path cannot be found because the structure document isn't updated, the Coder must **STOP** immediately.
3. **One-way Visibility**: The Coder is unaware of the "big goal" and only executes "atomic instructions" to prevent introducing systemic bugs by trying to "make things work" for the big goal.
4. **Snapshot is Truth**: When `LOGS` details conflict with the `SNAPSHOT`, the `SNAPSHOT` prevails.

## V. Cost-aware Routing

Based on task difficulty and Token consumption balance, the following model allocation is recommended:

- **Planner (Advanced Brain)**: Use **Claude 3.5 Sonnet** (or 4.5) or **GPT-4o**. Requires extreme logical deduction capability.
- **Coder (Scalpel)**: Use **Gemini 2.0 Flash**. Instructions are clear; pursues speed and low-cost context.
- **Evaluator (Auditor)**: Use **Claude 3.5 Sonnet**. Keen at checking code Diff and compliance.
- **Archivist (Memory Officer)**: Use **Claude 3.5 Sonnet** or **DeepSeek-V3/R1**. Requires strong information aggregation and summarization skills to compress long logs into high-density conclusions.

---

## VI. Initialization & Quick Start Guide

### Prerequisites

Before starting, ensure you have:

1. **A new project repository** with basic scaffolding (Next.js, Django, Flutter, etc.)
2. **Access to AI models** aligned with the Cost-aware Routing (Section V)
3. **Basic stack knowledge** of the technology being used
4. **30-60 minutes** for initial setup
5. **Markdown editor** (VS Code, Obsidian, or similar)

### 🚀 Project Initialization Checklist (Do This Once)

#### Step 1: Create the Directory Structure

```bash
your-project/
├── _DOCS/                 # Long-term Memory
│   ├── 00_STRUCTURE.md
│   ├── 01_DB_SCHEMA.md
│   ├── 02_STYLE_GUIDE.md
│   ├── 03_SERVER_ACTIONS.md
│   ├── 04_TECH_STACK.md
│   ├── 05_PROJECT_SNAPSHOT.md
│   └── LOGS/
│       └── LOG(format).md
├── _TASK/                 # Working Memory
│   ├── _PLAN.md
│   ├── _INSTRUCTION.md
│   └── _RESULT.md (optional: for evaluation notes)
├── src/                   # Your actual source code
├── README.md
└── .gitignore
```

#### Step 2: Initialize Long-term Memory (_DOCS)

**Create `_DOCS/00_STRUCTURE.md`:**
- Map the **actual directory structure** of your codebase
- List all key folders and their purposes
- Include technology/frameworks used
- Example: [See example in workspace structure]

**Create `_DOCS/01_DB_SCHEMA.md`:**
- Define your database schema (tables, fields, types, relationships)
- Note any constraints or indexes
- Include migration information if applicable
- This is the **source of truth** for backend data

**Create `_DOCS/02_STYLE_GUIDE.md`:**
- Document UI/UX conventions (colors, typography, spacing)
- Define interaction patterns (modals, forms, navigation)
- Include component naming conventions
- Link to design systems if available

**Create `_DOCS/03_SERVER_ACTIONS.md`:**
- List all API endpoints or server actions with:
  - HTTP method / Function name
  - Request/response schema
  - Authentication requirements
  - Error codes
- Format as a reference table or OpenAPI spec

**Create `_DOCS/04_TECH_STACK.md`:**
- List approved frameworks, libraries, and tools
- Define version constraints
- Note any deprecated or forbidden libraries
- Include setup/dev environment requirements

**Create `_DOCS/05_PROJECT_SNAPSHOT.md`:**
- High-level feature completion status
- Per-feature breakdown (**not** line-by-line code)
- Known issues and technical debt
- Key metrics (component count, test coverage, etc.)
- This is **maintained by the Archivist role**

**Create `_DOCS/LOGS/LOG(format).md`:**
- Template for development session logs
- Include: Goals, Completed Tasks, Issues, Focus for Next Session
- Naming convention: `YYYY-MM-DD.md` or `YYYY-MM-DD_TaskName.md`

#### Step 3: Initialize Working Memory (_TASK)

**Create `_TASK/_PLAN.md`:**

```markdown
# Strategy Board (The Plan)

## Roadmap
1. [ ] Phase 1: Core Feature A
2. [ ] Phase 2: Core Feature B
3. [ ] Phase 3: Integration & Polish

## CURRENT FOCUS
(Write your first task here in natural language)
- Example: "Build the user authentication flow with email/password"

## Notes / Scratchpad
- Any observations, decisions, or constraints
```

**Create `_TASK/_INSTRUCTION.md`:**

Leave this **empty or in standby mode** until the Planner phase populates it.

```markdown
# Task Instruction

## Status
**Awaiting Planner Assignment**

(This file will be populated when a CURRENT FOCUS is selected from _PLAN.md)
```

#### Step 4: Add to Git (Optional but Recommended)

```bash
git add _DOCS/ _TASK/
git commit -m "init: Initialize Dual-Brain System structure"
```

---

### ⚡ Standard Workflow: The P-C-E Loop (Repeated Until Feature Complete)

> **Key Concept**: The Planner → Coder → Evaluator cycle repeats **for each atomic task** until a complete component/feature is done. Only then does the **Archivist** run to consolidate everything.

```
┌─────────────────────────────────────────────────────────────────┐
│  ONE WORK LOOP (repeats until component is complete)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1️⃣ PLANNER: Define Task → _INSTRUCTION.md                    │
│  2️⃣ CODER: Execute Instructions                               │
│  3️⃣ EVALUATOR: Review & Log Results                           │
│                                                                  │
│  Decision Point:                                                │
│  ├─ Component done? → Go to ARCHIVIST                          │
│  └─ More tasks? → Repeat (back to PLANNER with next task)     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────────────────────────┐
│  ARCHIVIST (runs ONCE per component completion)                │
│  - Consolidate logs → update _SNAPSHOT.md                      │
│  - Clean up _PLAN.md                                            │
│  - Archive old logs                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

#### **Phase 1: Planner** 🧠

**When**: Before each development task
**Who**: You (the human operator) or a strategic AI agent
**Duration**: 5-10 minutes

**What to do**:

1. **Read** `_PLAN.md` and identify the next **CURRENT FOCUS** task
2. **Check** `_SNAPSHOT.md` to understand completed work and blockers
3. **Review** relevant `_DOCS/` files (structure, schema, API spec, style guide)
4. **Generate** `_INSTRUCTION.md` containing:
   - Clear **Context** (WHY this task matters)
   - Explicit **Context Scope** (≤4 files to modify)
   - Numbered, imperative **Steps** (WHAT to do, not HOW)
   - **Out of Scope** section (what NOT to touch)
   - **Constraints & Rules** (patterns, tools, requirements)
   - **Quality Checklist** (how to verify success)

**Example Output** (_INSTRUCTION.md):
```markdown
# Task Instruction: Implement User Registration Form

## Context
Enable new users to create accounts. This unblocks the full authentication flow.
Derived from: _PLAN.md - Phase 1: Authentication

## Context Scope (Strict)
- `src/components/Auth/RegisterForm.tsx`
- `src/actions/user.ts`
- `prisma/schema.prisma` (schema updates only)

## Steps (Execution Order)
1. Extend Prisma User model with `emailVerified` and `verificationCode` fields.
2. Create RegisterForm component with email/password validation.
3. Add `registerUser` server action with hashing and DB storage.
4. Return success/error message to frontend.

## Out of Scope
- Do not implement email verification yet.
- Do not create the login page.
- Do not modify authentication middleware.

## Constraints & Rules
- Use bcrypt for password hashing (already in `04_TECH_STACK.md`).
- Validate email format before DB insert.
- No console.logs in production code.

## Quality Checklist
- [ ] Form validation works for invalid email.
- [ ] Passwords are hashed before storage.
- [ ] Error messages are user-friendly.
- [ ] Code follows patterns in existing codebase.
```

---

#### **Phase 2: Coder** 🔧

**When**: After _INSTRUCTION.md is ready
**Who**: Implementation AI agent or you (the developer)
**Duration**: 30 minutes - 2 hours (depending on task scope)

**What to do**:

1. **Read ONLY** `_INSTRUCTION.md` (do NOT read `_PLAN.md` or `_SNAPSHOT.md`)
2. **Check** the referenced files in `_DOCS/` for context (schemas, style guide, API spec)
3. **Execute** the steps in order, modifying **ONLY files in Context Scope**
4. **Test** locally to ensure all steps work as expected
5. **Stop immediately** if you need a file outside of Context Scope (report to Planner)
6. **Report back** with a summary of changes and any blockers

**Critical Rules**:
- ❌ Do NOT read `_PLAN.md` (prevents scope creep and bias)
- ❌ Do NOT modify files outside of Context Scope
- ❌ Do NOT "improve" or "optimize" beyond the instructions
- ✅ Do ask for clarification if instructions are ambiguous
- ✅ Do test against Quality Checklist before reporting completion

---

#### **Phase 3: Evaluator** 🔍

**When**: After Coder reports completion
**Who**: You, another developer, or an auditing AI agent
**Duration**: 10-20 minutes

**What to do**:

1. **Review** the code changes (run `git diff` to see all modifications)
2. **Verify** that `_INSTRUCTION.md` was followed exactly
3. **Check** that NO files were modified outside the Context Scope
4. **Run** the code:
   - Execute tests related to the feature
   - Manual testing (happy path + edge cases)
   - Check for integration issues with existing code
5. **Record findings** in `_DOCS/LOGS/YYYY-MM-DD.md`:
   - Goals and completed tasks ✅
   - Issues or bugs discovered ⚠️
   - What to work on next 🎯
   - Any technical decisions made

**Example Log** (_DOCS/LOGS/2026-02-10.md):
```markdown
# Daily Log: 2026-02-10

## Goals
- Implement user registration form and validation.

## Completed Tasks
- [x] Extended Prisma User model with emailVerified field.
- [x] Created RegisterForm.tsx with email/password validation.
- [x] Implemented `registerUser` server action with bcrypt hashing.
- [x] Manual testing passed for happy path and validation errors.

## Issues & Blockers
- Email verification OTP not yet integrated (intentionally out of scope).
- Noticed form needs padding on mobile (minor UX issue, not blocking).

## Focus for Next Task
- Implement login form (similar structure to registration).
- Then: Add email verification flow.

## Technical Decisions
- Used bcrypt with saltRounds=10 per security best practices.
```

---

#### **Decision After Evaluation** 🔄

After the Evaluator finishes:

- **✅ If feature is COMPLETE**: Jump to **Archivist Phase** (see below)
- **❌ If more tasks are needed**: Go back to **Phase 1 (Planner)** and select the next CURRENT FOCUS task

**Example**:
```
Evaluator: "Registration form is done. Ready for login?"
Planner: "Yes. Now focusing on: Implement user login form."
└─ Generate new _INSTRUCTION.md → Coder executes → Evaluator reviews → Repeat until authentication feature is complete
```

---

#### **Phase 4: Archivist** 📚

**When**: **ONLY after a complete component/feature is done** (after multiple P-C-E loops)
**Who**: You or an AI Archivist
**Duration**: 15-30 minutes
**Frequency**: Once per feature completion (e.g., after Auth, after Cart, after Orders, etc.)

**Trigger Examples**:
- ✅ "Authentication feature complete" (Registration + Login + Verification done)
- ✅ "Shopping cart feature complete" (Add/Remove + Discount + Checkout done)
- ❌ NOT after every single task (too much overhead)

**What to do**:

1. **Compress knowledge**: Extract key decisions and patterns from `LOGS/` and merge into `05_PROJECT_SNAPSHOT.md`
2. **Update roadmap**: Mark completed items in `_PLAN.md` as done and clean up old entries
3. **Reset instruction**: Clear `_INSTRUCTION.md` back to standby state
4. **Consolidate docs**: Move any new discoveries into `_DOCS/` (schema changes, new patterns, etc.)
5. **Archive logs**: Move old daily logs into a history folder or delete if already consolidated

**Example** (Before Archivist runs):
```
_PLAN.md:
- [x] Phase 1.1: User Registration (DONE)
- [x] Phase 1.2: User Login (DONE)
- [x] Phase 1.3: Email Verification (DONE)
- [ ] Phase 2: Shopping Cart

LOGS/:
- 2026-02-07.md: Registration completed
- 2026-02-08.md: Login completed
- 2026-02-09.md: Discovered email validation bug
- 2026-02-10.md: Bug fixed, verification done
```

**After Archivist runs**:
```
_PLAN.md:
- [ ] Phase 2: Shopping Cart

_SNAPSHOT.md (updated with):
- [x] Phase 1: User Authentication - COMPLETE ✅
  - Registration: email/password signup with validation
  - Login: session-based auth with secure tokens
  - Email verification: OTP-based confirmation
  - Known Issue: Rate limiting needed for OTP requests (Phase 3 backlog)
  - Test Coverage: 87% for auth flows

LOGS/:
- LOG(format).md (template only)
- (old daily logs removed or archived)
```

---

### 📋 Concrete Example: Building Your First Feature

Let's build a **food ordering system** starting with **Restaurant Listing**.

---

**ITERATION 1: Component Setup**

```
🧠 PLANNER PHASE:
   Update _PLAN.md CURRENT FOCUS:
   "Build restaurant listing page - display all restaurants with name, rating, cuisines"
   
   Create _INSTRUCTION.md with scope:
   - src/pages/restaurants.tsx
   - src/actions/restaurant.ts
   - src/components/RestaurantCard.tsx

🔧 CODER PHASE:
   ✅ Create RestaurantCard component
   ✅ Implement getRestaurants() action
   ✅ Build page layout with grid
   ✅ Local testing passes

🔍 EVALUATOR PHASE:
   ✅ Code review passed
   ✅ Manual testing OK
   ✅ Log: "Component rendered. Ready for search/filter next."

❓ Decision: More restaurant feature work? YES → Keep looping
```

**ITERATION 2: Search & Filter**

```
🧠 PLANNER PHASE:
   Update CURRENT FOCUS: "Add search and cuisine filter to restaurant listing"
   
   Create _INSTRUCTION.md with scope (different task, same files):
   - src/components/RestaurantFilter.tsx (NEW)
   - src/actions/restaurant.ts (MODIFY)
   - src/pages/restaurants.tsx (MODIFY)

🔧 CODER PHASE:
   ✅ Create RestaurantFilter component
   ✅ Add filter logic to getRestaurants()
   ✅ Update page to use filter state
   ✅ Local testing passes

🔍 EVALUATOR PHASE:
   ✅ Code review passed
   ✅ Manual testing OK
   ✅ Log: "Search and filter working. Restaurant feature is COMPLETE."

❓ Decision: Feature complete? YES → Run ARCHIVIST
```

**ARCHIVIST PHASE: Consolidate & Move On**

```
📚 ARCHIVIST:
   ✅ Update _SNAPSHOT.md:
      [x] Restaurant Listing - COMPLETE
      - Display all restaurants
      - Search by name
      - Filter by cuisine
      
   ✅ Update _PLAN.md:
      Remove completed restaurant tasks
      Mark "Phase 1: Restaurant Listing" as DONE
      
   ✅ Archive LOGS:
      Move 2026-02-07.md and 2026-02-08.md → (old logs removed)
      
   ✅ Clean up _INSTRUCTION.md → standby state

😊 Ready for next feature: "Phase 2: Shopping Cart"
```

---

### 🛠️ Quick Reference: When to Use Each File

| Question | Answer Location | Who Reads |
| --- | --- | --- |
| "Where's the login API?" | `03_SERVER_ACTIONS.md` | Planner, Coder |
| "What fields does User table have?" | `01_DB_SCHEMA.md` | Everyone |
| "Can I use Vue.js?" | `04_TECH_STACK.md` | Planner |
| "What's a completed feature?" | `05_PROJECT_SNAPSHOT.md` | Planner, Archivist |
| "What happened in the last session?" | `LOGS/YYYY-MM-DD.md` | Evaluator, Archivist |
| "What should I build next?" | `_PLAN.md` (CURRENT FOCUS) | Planner |
| "Exactly how do I build it?" | `_INSTRUCTION.md` | Coder |

---

### ⚠️ Common Pitfalls & Solutions

| Pitfall | Symptom | Fix |
| --- | --- | --- |
| **Scope Creep** | _INSTRUCTION.md is too large or vague | Cap Context Scope at 4 files; use imperative language |
| **Hallucinating Paths** | Coder tries to edit non-existent files | Always list paths in 00_STRUCTURE.md first |
| **Lost Context** | Coder reads _PLAN.md and deviates | Strictly enforce "Coder reads ONLY _INSTRUCTION.md" |
| **Stale Snapshot** | Conflicts between LOGS and _SNAPSHOT.md | Run Archivist phase regularly (weekly minimum) |
| **Token Bloat** | Long, unfocused instructions | Trim instructions to atomic, single-feature tasks |
| **No Documentation** | Hard to onboard another AI/developer | Update _DOCS/ **immediately** after structural changes |

---