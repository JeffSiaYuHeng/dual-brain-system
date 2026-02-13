# Dependency Graph Setup Guide

## Overview

The dependency graph feature automatically maps import/export relationships in your codebase, helping the Planner make informed decisions about file modifications and their potential side-effects.

## Quick Start

### 1. Install Dependencies (Optional but Recommended)

For the most accurate analysis, install `madge`:

```bash
npm install -g madge
# or
npm install --save-dev madge
```

**Note**: The script includes a fallback analyzer if madge is not available, but madge provides more accurate results.

### 2. Generate the Dependency Graph

```bash
# Using npm
npm run gen:graph

# Or directly with node
node scripts/generate-dependency-graph.js
```

### 3. Include in Pre-Planning Workflow

Before each planning session, regenerate both structure and dependency graph:

```bash
npm run gen:all
# This runs both gen:structure and gen:graph
```

## How It Works

### Analysis Process

1. **Scans Source Directories**: `app`, `components`, `lib`, `utils`, `hooks`, `context`
2. **Detects Imports**: Analyzes `import` statements in `.ts`, `.tsx`, `.js`, `.jsx` files
3. **Builds Dependency Map**: Creates both forward (A imports B) and reverse (B is imported by A) maps
4. **Identifies High-Impact Files**: Ranks files by number of importers
5. **Generates AI-Friendly Summary**: Creates markdown with examples and usage guidelines

### Output Structure

The generated `_DOCS/06_DEPENDENCY_GRAPH.md` contains:

- **Statistics**: Overview of files and dependencies analyzed
- **High-Impact Files**: Top 20 most-imported files with key importers
- **Full Dependency Map**: Complete import graph (expandable JSON)
- **Reverse Dependency Map**: Who imports what (expandable JSON)
- **Usage Instructions**: How the Planner should use this data

## Integration with Dual-Brain System

### For Planners

The Planner skill now **requires** checking the dependency graph before defining Context Scope:

```md
Phase 2: Tactical Blueprint
1. Select Target task
2. Isolate the task
3. ✨ Analyze Dependencies (MANDATORY)
   - Check 06_DEPENDENCY_GRAPH.md
   - Identify importers
   - Assess side-effect risk
4. Define Context Scope + Reference Scope
```

### Decision Matrix

| Importers | Risk Level | Action |
|-----------|-----------|--------|
| 0-2 | Low | Standard Context Scope |
| 3-5 | Medium | Add warning in instruction |
| 6-10 | High | Add key importers to Reference Scope |
| 10+ | Critical | Consider breaking task or expanding Context Scope |

### Example Workflow

**Task**: Modify `lib/utils/validation.ts`

**Planner's Steps**:
1. Read `06_DEPENDENCY_GRAPH.md`
2. Find: `validation.ts` imported by 12 files
3. Identify top importers:
   - `app/auth/login/page.tsx`
   - `components/forms/UserForm.tsx`
   - `lib/actions/user-actions.ts`
4. Add to `_INSTRUCTION.md`:
   ```md
   ## Context Scope
   - lib/utils/validation.ts

   ## Reference Scope (Read-Only)
   - app/auth/login/page.tsx (uses validateEmail)
   - components/forms/UserForm.tsx (uses validatePassword)

   ## Note
   ⚠️ High-impact file: Imported by 12 files. Test affected components after changes.
   ```

### For Evaluators

The Evaluator should verify:
- If modifying a high-impact file, were affected importers considered?
- Did the Coder test key importing files?
- Were breaking changes documented in `03_SERVER_ACTIONS.md`?

## Automation Options

### Git Hook (Pre-Planning)

Create `.git/hooks/pre-commit` (if using git-based planning workflow):

```bash
#!/bin/sh
# Auto-regenerate dependency graph before commits
npm run gen:graph
git add _DOCS/06_DEPENDENCY_GRAPH.md
```

### CI/CD Integration

Add to your CI pipeline to keep docs updated:

```yaml
# .github/workflows/update-docs.yml
name: Update Documentation
on: [push]
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install -g madge
      - run: npm run gen:all
      - run: git diff --exit-code || (git add _DOCS/*.md && git commit -m "docs: auto-update" && git push)
```

### Manual Trigger

Add to `package.json` scripts:

```json
{
  "scripts": {
    "pre-plan": "npm run gen:all"
  }
}
```

## Configuration

### Customize Analyzed Directories

Edit `scripts/generate-dependency-graph.js`:

```javascript
const SOURCE_DIRS = [
  'app',
  'components',
  'lib',
  'utils',
  'hooks',
  'context',
  'services',  // Add custom directories
  'api'
];
```

### Adjust High-Impact Threshold

```javascript
.slice(0, 20); // Top 20 → Change to show more/fewer
```

### File Extensions

```javascript
const VALID_EXTENSIONS = ['.ts', '.tsx', '.js', '.jsx', '.mjs'];
```

## Troubleshooting

### Issue: "madge not found"

**Solution**: The script will use the built-in fallback analyzer. For better accuracy:
```bash
npm install -g madge
```

### Issue: "No files analyzed"

**Solution**: Check if source directories exist:
```bash
ls app/ components/ lib/ utils/
```

If directories don't exist, update `SOURCE_DIRS` in the script.

### Issue: "Import paths not resolved"

**Solution**: The script supports:
- Relative imports: `./utils/foo`
- Alias imports: `@/components/Bar`

If using custom aliases, update `resolveImportPath()` function.

## Benefits

✅ **100% Accurate**: Based on actual code, not guessing
✅ **Zero Token Cost**: Pre-generated, no LLM scanning needed
✅ **Prevents Breaking Changes**: See downstream effects before modifying
✅ **Smarter Scope Decisions**: Data-driven Context/Reference Scope
✅ **Faster Debug Cycles**: Fewer "forgot to update caller" bugs
✅ **Enables Refactoring**: Understand blast radius of changes

## Next Steps

1. ✅ Install madge (optional)
2. ✅ Run `npm run gen:graph`
3. ✅ Verify `_DOCS/06_DEPENDENCY_GRAPH.md` was created
4. ✅ Add to pre-planning workflow
5. ✅ Update Planner to always check dependency graph

---

**Related Documentation:**
- `_DOCS/06_DEPENDENCY_GRAPH.md` - Generated dependency map
- `.agent/skills/dual-brain-planner/SKILL.md` - Planner workflow
- `.agent/skills/dual-brain-evaluator/SKILL.md` - Evaluation criteria
