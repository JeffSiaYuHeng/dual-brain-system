# Skills Update Summary

**Date**: 2026-02-13
**Purpose**: Document all skill updates to reflect new documentation structure and features

---

## Overview of Changes

The dual-brain system has been enhanced with three major features:
1. **Reference Scope** (Read-Only Context) - Planner feature
2. **Debug Mode** (Short-Circuit Loop: E → C) - Evaluator feature
3. **Dependency Graph** (Import/Export Analysis) - System-wide feature
4. **Cross-Reference & Impact Audit** - Archivist feature

All agent skills have been updated to support these new capabilities.

---

## 📊 New Documentation Structure

### Updated _DOCS/ Contents

| File | Description | Generator |
|------|-------------|-----------|
| `00_STRUCTURE.md` | File map | `npm run gen:structure` |
| `01_DB_SCHEMA.md` | Database schema | Manual |
| `02_STYLE_GUIDE.md` | UI/UX guidelines | Manual |
| `03_SERVER_ACTIONS.md` | API protocol + Side Effects | Manual + Archivist |
| `04_TECH_STACK.md` | Tech stack | Manual |
| `05_PROJECT_SNAPSHOT.md` | Project state | Archivist |
| `06_DEPENDENCY_GRAPH.md` | Import/export map | `npm run gen:graph` |

### New _TASK/ Files

| File | Purpose | Created By |
|------|---------|------------|
| `_INSTRUCTION.md` | Normal execution blueprint | Planner |
| `_FIX_INSTRUCTION.md` | Debug mode quick fix | Evaluator |
| `_PLAN.md` | Strategic roadmap | Planner (updated by Evaluator) |

---

## 🔄 Skill-by-Skill Updates

### 1. Planner Skill (dual-brain-planner)

**New Capabilities:**
- ✅ Must consult `06_DEPENDENCY_GRAPH.md` before defining Context Scope
- ✅ Can specify **Reference Scope** (1-2 read-only files) in instructions
- ✅ Must analyze high-impact files and their importers
- ✅ Uses decision matrix to determine scope based on importer count

**New Sections Added:**
- `🔗 Dependency Analysis (Critical for Scope Definition)` - Complete guide on using dependency graph
- Updated Read Access to include `06_DEPENDENCY_GRAPH.md`
- Updated Phase 2 workflow to mandate dependency analysis
- Updated instruction template to include Reference Scope section

**Decision Matrix:**
| Importers | Risk | Action |
|-----------|------|--------|
| 0-2 | Low | Standard Context Scope |
| 3-5 | Medium | Add warning note |
| 6-10 | High | Add key importers to Reference Scope |
| 10+ | Critical | Break task or expand Context Scope |

**Example Workflow:**
```md
Task: Modify lib/utils/validation.ts

1. Check 06_DEPENDENCY_GRAPH.md
2. Found: Imported by 12 files
3. Decision: Add top 2 importers to Reference Scope

Result in _INSTRUCTION.md:
  Context Scope:
    - lib/utils/validation.ts

  Reference Scope (Read-Only):
    - app/auth/login/page.tsx
    - components/forms/UserForm.tsx
```

---

### 2. Coder Skill (dual-brain-coder)

**New Capabilities:**
- ✅ Can process `_FIX_INSTRUCTION.md` for Debug Mode fixes
- ✅ Can read Reference Scope files (read-only) for context
- ✅ Understands distinction between Context Scope (writable) and Reference Scope (read-only)
- ✅ Follows quick-fix protocol in Debug Mode

**New Sections Added:**
- `📖 Understanding Reference Scope (Read-Only Context)` - How to use read-only context files
- `🔧 Debug Mode (Quick Fix Protocol)` - Emergency fix workflow from Evaluator
- Updated Execution Rules to check for both instruction types
- Updated Failure Conditions to include Reference Scope violations

**Instruction Selection Logic:**
```
1. Check if _TASK/_FIX_INSTRUCTION.md exists
   → Yes: Debug Mode (quick fix only)
   → No: Normal Mode (full feature implementation)

2. In either mode:
   - Context Scope = files you can MODIFY
   - Reference Scope = files you can READ (but NOT modify)
```

**Debug Mode Rules:**
- ❌ No feature additions
- ❌ No refactoring
- ❌ No "fixing" other issues
- ✅ ONLY fix the specific error in `_FIX_INSTRUCTION.md`

---

### 3. Evaluator Skill (dual-brain-evaluator)

**New Capabilities:**
- ✅ Can activate **Debug Mode** for simple errors (syntax, imports, typos)
- ✅ Can create `_FIX_INSTRUCTION.md` directly for Coder
- ✅ Can verify dependency graph awareness in scope decisions
- ✅ Maximum 2 Debug Mode iterations before escalating to Planner

**New Sections Added:**
- `🔧 Debug Mode (Short-Circuit Loop: E → C)` - Complete Debug Mode workflow
- Qualifying vs non-qualifying errors for Debug Mode
- `_FIX_INSTRUCTION.md` template
- Debug Mode exit conditions
- Updated Final Handover to include Debug Mode path
- Updated logging template with Debug Mode status

**Qualifying Errors for Debug Mode:**
- ✅ Syntax Error (missing semicolon, bracket, quote)
- ✅ Import Missing (wrong import path)
- ✅ Typo (variable name mismatch)
- ✅ Type Error (simple type annotation)
- ✅ Linting Error (unused variable)

**NOT Qualifying (Must escalate to Planner):**
- ❌ Logic errors
- ❌ Schema violations
- ❌ Architectural problems
- ❌ Missing features
- ❌ Multiple unrelated errors

**Debug Mode Workflow:**
```
1. Detect qualifying error during build check
2. Create _TASK/_FIX_INSTRUCTION.md
3. Log error with status "FAILED (Debug Mode Activated)"
4. Handover to Coder (bypass Planner)
5. After fix: Return to normal evaluation workflow
```

---

### 4. Archivist Skill (dual-brain-archivist)

**New Capabilities:**
- ✅ Must perform **Cross-Reference & Impact Audit** on `03_SERVER_ACTIONS.md`
- ✅ Analyzes logic changes for implicit side-effects
- ✅ Documents side effects explicitly in API documentation
- ✅ Maintains API contract integrity to prevent frontend hallucination

**New Sections Added:**
- `Cross-Reference & Impact Audit (深度关联与副作用分析)` - Complete audit workflow
- Logic Tracing: Identify cascading effects
- Side Effect Documentation: Explicit annotations
- Boundary Synchronization: Breaking change protocol
- Updated Data Access to include write access to `03_SERVER_ACTIONS.md`
- Updated Defensive Rules with API Contract Integrity requirements

**Impact Audit Workflow:**
```
Step 3 (NEW): Cross-Reference & Impact Audit

1. Logic Tracing:
   - Review completed tasks
   - Identify implicit cross-effects
   - Example: "Does updating user balance trigger notifications?"

2. Side Effect Documentation:
   - Add/update Side Effects sections in 03_SERVER_ACTIONS.md
   - Format:
     ### updateUserProfile
     **Side Effects**:
     - ⚠️ Changing `email` triggers email verification workflow
     - ⚠️ Updating `role` invalidates cached permissions

3. Boundary Synchronization:
   - If return structure changed → Update API docs immediately
   - Add ⚠️ BREAKING CHANGE annotation with date
   - Mark affected frontend components
```

**Side Effect Annotation Example:**
```md
**⚠️ BREAKING CHANGE (2026-02-13)**:
- Return type changed from `{success: boolean}` to `{success: boolean, userId: string}`
- Frontend components using this action must be updated
```

---

## 🔗 Integration Between Skills

### Normal Flow (P → C → E → A)

```
1. Planner:
   - Reads 06_DEPENDENCY_GRAPH.md
   - Identifies high-impact files
   - Defines Context Scope + Reference Scope
   - Creates _INSTRUCTION.md

2. Coder:
   - Reads _INSTRUCTION.md
   - Reads Reference Scope files (context only)
   - Modifies Context Scope files
   - Reports completion

3. Evaluator:
   - Runs build verification
   - Checks scope adherence
   - Verifies dependency awareness
   - Creates log entry
   - Ticks checkbox in _PLAN.md

4. Archivist (when triggered):
   - Performs impact audit
   - Updates 03_SERVER_ACTIONS.md with side effects
   - Compresses logs
   - Updates PROJECT_SNAPSHOT.md
```

### Debug Flow (E → C → E)

```
1. Evaluator:
   - Detects simple error (syntax, import, typo)
   - Creates _FIX_INSTRUCTION.md
   - Logs "FAILED (Debug Mode Activated)"
   - Bypasses Planner

2. Coder:
   - Reads _FIX_INSTRUCTION.md
   - Applies ONLY the specified fix
   - Runs verification
   - Reports result

3. Evaluator:
   - Re-runs build verification
   - If success: Continue normal workflow
   - If still failing: Escalate to Planner
   - If new error: Max 2 iterations, then escalate
```

---

## 📋 Verification Checklist

Use this checklist to verify all skills are properly updated:

### Planner
- [x] Read access includes `06_DEPENDENCY_GRAPH.md`
- [x] Phase 2 workflow includes mandatory dependency analysis
- [x] Instruction template includes Reference Scope section
- [x] Quality checklist verifies Reference Scope ≤ 2 files
- [x] Decision matrix provided for importer counts

### Coder
- [x] Allowed inputs include `_FIX_INSTRUCTION.md`
- [x] Allowed inputs include Reference Scope (read-only)
- [x] Absolute restrictions include Reference Scope modification ban
- [x] Execution rules check for both instruction types
- [x] Debug Mode protocol documented
- [x] Reference Scope usage guide included
- [x] Post-execution handover includes Debug Mode path

### Evaluator
- [x] Debug Mode section complete with qualifying errors
- [x] `_FIX_INSTRUCTION.md` template provided
- [x] Debug Mode exit conditions defined
- [x] Final handover includes Debug Mode path
- [x] Logging template includes Debug Mode status
- [x] Integration notes mention dependency graph verification

### Archivist
- [x] Data access includes `03_SERVER_ACTIONS.md` write permission
- [x] Cross-Reference & Impact Audit workflow added
- [x] Logic tracing process documented
- [x] Side effect documentation format provided
- [x] Boundary synchronization protocol defined
- [x] Defensive rules include API Contract Integrity

---

## 🚀 Benefits of Updated System

### 1. Dependency Graph Benefits
- ✅ **100% Accurate**: Based on actual code, not guessing
- ✅ **Zero Token Cost**: Pre-generated, no LLM scanning
- ✅ **Prevents Breaking Changes**: See downstream effects before modifying
- ✅ **Smarter Scoping**: Data-driven Context/Reference Scope decisions
- ✅ **Faster Debug Cycles**: Fewer "forgot to update caller" bugs

### 2. Reference Scope Benefits
- ✅ **Better Context**: Coder sees how functions are used
- ✅ **Reduced Errors**: Implementation matches actual usage patterns
- ✅ **Clear Boundaries**: Explicit read-only vs writable distinction
- ✅ **Safer Refactoring**: Can see callers without modifying them

### 3. Debug Mode Benefits
- ✅ **Faster Fixes**: Skip Planner for simple errors
- ✅ **Reduced Latency**: Direct E → C → E loop
- ✅ **Focused Repairs**: Only fix the specific error
- ✅ **Automatic Escalation**: Complex issues still go to Planner

### 4. Impact Audit Benefits
- ✅ **API Integrity**: Frontend never uses outdated contracts
- ✅ **Explicit Side Effects**: No hidden coupling
- ✅ **Change Tracking**: Breaking changes clearly documented
- ✅ **Prevents Hallucination**: Truth files stay synchronized

---

## 📚 Related Documentation

- **`DEPENDENCY_GRAPH_SETUP.md`** - Complete setup guide for dependency analysis
- **`README.md`** - Updated with new features and documentation structure
- **`package.json`** - New scripts: `gen:graph`, `gen:all`, `pre-plan`
- **`scripts/generate-dependency-graph.js`** - Dependency graph generator
- **`scripts/generate-structure.js`** - Structure map generator

---

## 🔄 Migration Guide

### For Existing Projects

1. **Install Dependencies**:
   ```bash
   npm install -g madge  # Optional but recommended
   ```

2. **Generate Documentation**:
   ```bash
   npm run gen:all
   ```

3. **Update Skills**:
   - All skills have been automatically updated
   - Review skill definitions in `.agent/skills/*/SKILL.md`

4. **Update Existing Instructions**:
   - Add Reference Scope to existing `_INSTRUCTION.md` files
   - Consult `06_DEPENDENCY_GRAPH.md` before creating new instructions

5. **Configure Pre-Planning Hook**:
   ```bash
   # Add to your workflow
   npm run pre-plan  # Runs gen:all before planning
   ```

### For New Projects

1. **Clone/Copy Structure**:
   ```bash
   cp -r _DOCS/ _TASK/ .agent/ scripts/ <your-project>/
   cp package.json <your-project>/
   ```

2. **Initialize Documentation**:
   ```bash
   npm run gen:all
   # Then manually create: 01_DB_SCHEMA.md, 02_STYLE_GUIDE.md, etc.
   ```

3. **Start Planning**:
   - Use dual-brain-planner skill
   - System will automatically use dependency graph
   - Reference Scope will be available

---

## 🎯 Next Steps

1. ✅ All skills updated
2. ✅ Documentation structure enhanced
3. ✅ Automation scripts created
4. ⏭️ Test the system with a real task
5. ⏭️ Verify Debug Mode works correctly
6. ⏭️ Validate dependency graph accuracy
7. ⏭️ Monitor for edge cases and refine

---

**Version**: 2.1
**Status**: ✅ Complete
**Last Updated**: 2026-02-13
