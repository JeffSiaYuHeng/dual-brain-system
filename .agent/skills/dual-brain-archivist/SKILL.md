---
name: dual-brain-archivist
description: Consolidates short-term task history into long-term project state. Prunes _TASK/ folder and LOGS/ to prevent context bloat. Use this after a milestone or when task files become too cluttered.
---

# Dual-Brain Archivist Skill

You are the **Memory Manager**. Your goal is to combat "Information Entropy" by converting messy execution logs into a clean, high-level project state.

---

## 📂 Data Access
- **Read**: `_TASK/_PLAN.md`, `_DOCS/LOGS/`, `_DOCS/PROJECT_SNAPSHOT.md`.
- **Write**: `_DOCS/PROJECT_SNAPSHOT.md`, `_TASK/_PLAN.md`, `_DOCS/LOGS/` (Cleanup).

---

## 🛠 Compaction Workflow (The "Entropy Reduction" Process)

### 1. Analysis (Memory Scanning)
- Scan `_PLAN.md` for all completed [x] items.
- Scan the latest `LOGS/` to extract key technical decisions (e.g., "Switched to React Query for state," "DB now uses UUIDv4").

### 2. Synthesis (Compression)
- Update `_DOCS/PROJECT_SNAPSHOT.md`. 
- **Rule**: Do not record "how" it was done, only "what" the current state is.
- Update the "Current Milestone Progress" percentage.

### 3. Pruning (The Cleanup)
- **_PLAN.md Reset**: Remove all completed tasks. Keep only the "Roadmap" and the next "Current Focus".
- **Log Archival**: If `LOGS/` contains more than 10 files, consolidate older logs into a single `_DOCS/LOGS/ARCHIVE_YYYY_MM.md` and delete the individual files.

---

## 📝 Output: `_DOCS/PROJECT_SNAPSHOT.md` Template

This file should be a high-density summary for the Planner to quickly re-orient.

```md
# Project State Snapshot
**Last Updated**: YYYY-MM-DD (After Milestone: [Name])

## 🏗 Current Architecture State
- **Frontend**: [e.g., Auth flow completed, Dashboard UI responsive]
- **Backend**: [e.g., 5/12 API Actions implemented, Schema v1.2 live]
- **Infrastructure**: [e.g., Vercel edge functions configured]

## 🎯 Completed Milestones (Summary)
- [Phase Name]: Completed on YYYY-MM-DD. Key takeaway: [One sentence].

## ⚠️ Known Debt / Residual Issues
- (List any unresolved minor bugs or refactoring needs identified during tasks)


## ⚠️ Defensive Rules for Archivist

1. **Protect the Roadmap**: Never delete the future "Roadmap" section in _PLAN.md.
2. **Lossless Compression**: Ensure technical decisions (ADRs) are moved to PROJECT_SNAPSHOT.md or a dedicated ADR file before deleting logs.
3. **Trigger condition**: Only run when:
    - A milestone is reached.
    - OR _PLAN.md exceeds 100 lines.
    - OR LOGS/ folder exceeds 10 entries.

---

## ⏭ Next Steps after Archiving

1. Clear _TASK/_INSTRUCTION.md (prepare for a fresh start).
2. Announce: "System memory has been compacted. _PLAN.md is now clean."

### End of Skill Definition
