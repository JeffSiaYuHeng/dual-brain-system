# Dual-Brain System - Quick Reference Card

**Version 2.1** | Last Updated: 2026-02-13

---

## 🗂️ Documentation Files

| File | Purpose | Updated By | Command |
|------|---------|------------|---------|
| `00_STRUCTURE.md` | File tree | Auto | `npm run gen:structure` |
| `01_DB_SCHEMA.md` | Database schema | Manual | - |
| `02_STYLE_GUIDE.md` | UI/UX rules | Manual | - |
| `03_SERVER_ACTIONS.md` | API + Side Effects | Manual + Archivist | - |
| `04_TECH_STACK.md` | Tech stack | Manual | - |
| `05_PROJECT_SNAPSHOT.md` | Project state | Archivist | - |
| `06_DEPENDENCY_GRAPH.md` | Import/export map | Auto | `npm run gen:graph` |

---

## 🤖 Agent Roles

### Planner
**Role**: Strategy → Tactics
**Reads**: `_PLAN.md`, all `_DOCS/`, code
**Writes**: `_INSTRUCTION.md`
**New**: Must check dependency graph, can add Reference Scope

### Coder
**Role**: Minimal Execution
**Reads**: `_INSTRUCTION.md` or `_FIX_INSTRUCTION.md`, Context Scope, Reference Scope
**Writes**: Context Scope files only
**New**: Supports Debug Mode, respects Reference Scope (read-only)

### Evaluator
**Role**: Audit & Quality Gate
**Reads**: `_PLAN.md`, `_INSTRUCTION.md`, code diff
**Writes**: `LOGS/`, `_PLAN.md` (checkboxes), `_FIX_INSTRUCTION.md`
**New**: Can activate Debug Mode for simple errors

### Archivist
**Role**: Memory Compression
**Reads**: `_PLAN.md`, `LOGS/`, all `_DOCS/`
**Writes**: `PROJECT_SNAPSHOT.md`, `03_SERVER_ACTIONS.md`, cleans `_PLAN.md` and `LOGS/`
**New**: Performs impact audit, documents side effects

---

## 🔄 Workflows

### Normal Flow: P → C → E (→ A)

```
┌─────────────┐
│   Planner   │ 1. Check 06_DEPENDENCY_GRAPH.md
│             │ 2. Define Context Scope (≤4 files)
│             │ 3. Define Reference Scope (≤2 files, read-only)
│             │ 4. Create _INSTRUCTION.md
└──────┬──────┘
       │
       v
┌─────────────┐
│    Coder    │ 1. Read _INSTRUCTION.md
│             │ 2. Read Reference Scope (context)
│             │ 3. Modify Context Scope files
│             │ 4. Report completion
└──────┬──────┘
       │
       v
┌─────────────┐
│  Evaluator  │ 1. Run build check
│             │ 2. Verify scope adherence
│             │ 3. Check dependency awareness
│             │ 4. Create log + tick checkbox
└──────┬──────┘
       │
       v (if milestone reached)
┌─────────────┐
│  Archivist  │ 1. Perform impact audit
│             │ 2. Update 03_SERVER_ACTIONS.md
│             │ 3. Compress logs
│             │ 4. Update PROJECT_SNAPSHOT.md
└─────────────┘
```

### Debug Flow: E → C → E

```
┌─────────────┐
│  Evaluator  │ Error: Syntax/Import/Typo
│             │ ↓
│             │ 1. Detect qualifying error
│             │ 2. Create _FIX_INSTRUCTION.md
│             │ 3. Bypass Planner
└──────┬──────┘
       │
       v
┌─────────────┐
│    Coder    │ 1. Read _FIX_INSTRUCTION.md
│             │ 2. Apply ONLY the specified fix
│             │ 3. Run verification
│             │ 4. Report result
└──────┬──────┘
       │
       v
┌─────────────┐
│  Evaluator  │ 1. Re-audit
│             │ 2. If pass → Continue
│             │ 3. If fail → Escalate to Planner
│             │    (max 2 iterations)
└─────────────┘
```

---

## 📊 Scope Types

### Context Scope (Writable)
- **Max**: 4 files
- **Permission**: Read + Write
- **Purpose**: Files to be modified
- **Example**: `lib/utils/validation.ts`

### Reference Scope (Read-Only)
- **Max**: 2 files
- **Permission**: Read only
- **Purpose**: Context for understanding usage
- **Example**: `app/login/page.tsx` (calls validation functions)

### Out of Scope
- **Permission**: None
- **Purpose**: Explicitly forbidden files
- **Example**: `lib/auth/secret-handler.ts`

---

## 🎯 Decision Matrices

### Dependency Impact (Planner)

| Importers | Risk | Action |
|-----------|------|--------|
| 0-2 | Low | Standard Context Scope |
| 3-5 | Medium | Add warning note |
| 6-10 | High | Add 1-2 to Reference Scope |
| 10+ | Critical | Break task or expand scope |

### Debug Mode Eligibility (Evaluator)

| Error Type | Qualifies? | Action |
|------------|-----------|--------|
| Syntax error | ✅ Yes | Debug Mode |
| Missing import | ✅ Yes | Debug Mode |
| Typo | ✅ Yes | Debug Mode |
| Type annotation | ✅ Yes | Debug Mode |
| Logic error | ❌ No | Escalate to Planner |
| Schema violation | ❌ No | Escalate to Planner |
| Multiple errors | ❌ No | Escalate to Planner |

---

## 🚀 Quick Commands

```bash
# Generate structure map
npm run gen:structure

# Generate dependency graph
npm run gen:graph

# Generate both
npm run gen:all

# Pre-planning routine (recommended)
npm run pre-plan
```

---

## 📝 File Templates

### _INSTRUCTION.md (Planner Output)

```md
# Task Instruction

## Context
[Brief summary]

---

## Context Scope (Strict)
- path/to/file1.ts
- path/to/file2.tsx
(Max 4 files)

---

## Reference Scope (Read-Only)
- path/to/caller1.tsx (uses functionA)
- path/to/caller2.ts (uses functionB)
(Max 2 files)

---

## Steps (Execution Order)
1. [Step 1]
2. [Step 2]

---

## Constraints & Rules
- [Rule 1]
- [Rule 2]

---

## Out of Scope (Hard Stop)
- [Forbidden file 1]
- [Forbidden file 2]
```

### _FIX_INSTRUCTION.md (Evaluator Output)

```md
# Fix Instruction (Debug Mode)

## Error Type
[SYNTAX_ERROR / IMPORT_MISSING / TYPE_ERROR / TYPO / LINTING]

## Error Message
```
[Exact error from build]
```

## Target File
- path/to/file.ts:LINE_NUMBER

## Required Fix
[Single sentence: "Add missing import for X from Y"]

## Constraints
- DO NOT modify any other files
- DO NOT change logic
- ONLY fix the reported error

## Verification
After fix, run: `npm run build`
```

---

## ⚠️ Common Pitfalls

### Planner
- ❌ Ignoring dependency graph
- ❌ Adding >4 files to Context Scope
- ❌ Adding >2 files to Reference Scope
- ❌ Including code snippets in instructions

### Coder
- ❌ Modifying Reference Scope files
- ❌ Reading `_PLAN.md`
- ❌ Editing files outside Context Scope
- ❌ Adding features in Debug Mode

### Evaluator
- ❌ Ticking checkboxes before build passes
- ❌ Using Debug Mode for logic errors
- ❌ More than 2 Debug Mode iterations
- ❌ Not checking scope adherence

### Archivist
- ❌ Deleting the Roadmap section
- ❌ Archiving without checking API changes
- ❌ Forgetting to document side effects
- ❌ Running outside trigger conditions

---

## 🔍 Debugging Tips

### Build Fails
1. Check Evaluator log for exact error
2. If syntax/import/typo → Debug Mode
3. If logic/schema error → Back to Planner
4. Max 2 Debug Mode attempts, then escalate

### Scope Violations
1. Check `_INSTRUCTION.md` Context Scope
2. Verify Coder didn't touch Reference Scope
3. Check if files are in Out of Scope list
4. Review git diff vs. allowed files

### Dependency Issues
1. Check if file is in High-Impact section
2. Verify importers were considered
3. Check if Reference Scope was used
4. Run `npm run gen:graph` to refresh

### Side Effect Surprises
1. Check `03_SERVER_ACTIONS.md` for documented effects
2. Run Archivist to perform impact audit
3. Check if return types changed
4. Verify frontend contracts match backend

---

## 📚 Documentation Hierarchy

```
_DOCS/                     (Long-term Memory - Read-Only)
├── 00_STRUCTURE.md       ← Auto-generated
├── 01_DB_SCHEMA.md       ← Manual (Backend Truth)
├── 02_STYLE_GUIDE.md     ← Manual (UI Truth)
├── 03_SERVER_ACTIONS.md  ← Manual + Archivist (API Truth)
├── 04_TECH_STACK.md      ← Manual
├── 05_PROJECT_SNAPSHOT.md ← Archivist
├── 06_DEPENDENCY_GRAPH.md ← Auto-generated
└── LOGS/                 ← Evaluator writes, Archivist archives
    ├── 2026-02-13.md
    └── ARCHIVE_2026_01.md

_TASK/                    (Working Memory - High Churn)
├── _PLAN.md             ← Planner writes, Evaluator ticks
├── _INSTRUCTION.md      ← Planner writes, Coder reads
└── _FIX_INSTRUCTION.md  ← Evaluator writes, Coder reads
```

---

## 🎓 Best Practices

### Pre-Planning
✅ Always run `npm run pre-plan`
✅ Review recent logs before planning
✅ Check PROJECT_SNAPSHOT for current state
✅ Verify dependency graph is up-to-date

### Planning
✅ Start by checking dependency graph
✅ Use Reference Scope for high-impact files
✅ Keep Context Scope ≤ 4 files
✅ Use imperative verbs in steps

### Coding
✅ Read Reference Scope for context
✅ Never modify Reference Scope files
✅ Stop if instructions conflict with code
✅ Report completion, don't self-verify

### Evaluating
✅ Run structure sync + build check
✅ Verify scope adherence strictly
✅ Use Debug Mode only for simple errors
✅ Document all failures in logs

### Archiving
✅ Only run at milestones or when triggered
✅ Perform full impact audit on API changes
✅ Document all side effects explicitly
✅ Preserve Roadmap section in _PLAN.md

---

## 📞 Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| Path not found | Run `npm run gen:structure` |
| Breaking change | Check `03_SERVER_ACTIONS.md` Side Effects |
| Build fails repeatedly | Use Debug Mode (if simple) or escalate to Planner |
| Scope creep | Review `_INSTRUCTION.md` Context Scope |
| Unknown dependencies | Run `npm run gen:graph` |
| Memory overload | Run Archivist to compress logs |
| API mismatch | Run Archivist impact audit |

---

**Keep this card handy for quick reference during development!**

**Full Documentation**: See `README.md`, `SKILLS_UPDATE_SUMMARY.md`, and `DEPENDENCY_GRAPH_SETUP.md`
