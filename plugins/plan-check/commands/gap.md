---
description: Analyze what existing code might break and update the plan with mitigations
---

# Plan Check: Gap (Breakage Analysis)

Analyze how planned changes will affect existing code and update the plan with mitigations.

## Step 1: Breakage Analysis

Find the plan: glob `~/.claude/plans/*.md` and read the most recently modified file (use `stat` to compare). If no plan file found on disk, extract plan text from conversation context. Note the plan file path (or null if from conversation).

Launch the **breakage-analyst** agent, passing it the full plan text and the plan file path. The agent reads source files itself as needed, traces callers and importers, and builds its own caller map.

The agent will return findings in BRK-NNN format with plan amendments, including regression test recommendations.

## Step 2: Update Plan

1. Read the plan file path from Step 1
2. Collect all findings and plan amendments from Step 1
3. Apply ALL amendments (every severity level -- Low through Critical), processing in severity order (Critical first, then High, Medium, Low). If two amendments target the same section with incompatible operations, apply the higher-severity amendment and print a note flagging the conflict.
4. Add regression test items if the breakage-analyst recommended them
5. Append a "Breakage Analysis Notes" section listing all changes made, with finding IDs and severity levels
6. If the plan was from conversation context (no file path), write the amended plan to `~/.claude/plans/amended-plan.md` and report its path
7. Print a brief summary: count of findings by severity, confirmation that the plan was updated
