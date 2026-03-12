---
description: Add or improve an implementation checklist in the plan
---

# Plan Check: Checklist

Evaluate the plan's implementation checklist (or create one) and update the plan directly.

## Step 1: Checklist Generation

Find the plan: glob `~/.claude/plans/*.md` and read the most recently modified file (use `stat` to compare). If no plan file found on disk, extract plan text from conversation context. Note the plan file path (or null if from conversation).

Launch the **checklist-architect** agent, passing it the full plan text and the plan file path. The agent reads source files and searches for patterns itself as needed.

The agent will return findings in CHK-NNN format with plan amendments containing the complete checklist.

## Step 2: Update Plan

1. Read the plan file path from Step 1
2. Collect all findings and plan amendments from Step 1
3. Apply ALL amendments (every severity level -- Low through Critical), processing in severity order (Critical first, then High, Medium, Low). If two amendments target the same section with incompatible operations, apply the higher-severity amendment and print a note flagging the conflict. In practice, the checklist-architect typically emits a single `replace` or `append-section` amendment for the full checklist -- if the plan has an existing checklist section, replace it; if not, append the new checklist section.
4. Append a "Checklist Notes" section listing CHK findings with IDs, severity, and what was changed
5. If the plan was from conversation context (no file path), write the amended plan to `~/.claude/plans/amended-plan.md` and report its path
6. Print a brief summary: count of findings by severity, confirmation that the plan was updated
