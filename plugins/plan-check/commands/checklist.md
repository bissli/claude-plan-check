---
description: Validate and generate an implementation checklist for a plan
---

# Plan Check: Checklist

Evaluate and produce an implementation checklist for a plan.

## Step 1: Context Gathering (Haiku agent)

Launch a **Haiku** agent to gather project context. The agent should:

1. Read all `.claude/rules/*.md` files (glob for them first)
2. Read all `.claude/reference/*.md` files (glob for them first)
3. Read the root `CLAUDE.md` and any directory-level `CLAUDE.md` files found via glob `**/CLAUDE.md`
4. Find the plan:
   - Glob `~/.claude/plans/*.md` and read the most recently modified file (use `stat` to compare)
   - If no plan file found on disk, extract plan text from conversation context
5. From the plan, list all source files it references (paths mentioned in the plan text)

Return: all rules content, all CLAUDE.md content, the full plan text, a one-paragraph plan summary, and the list of referenced source files.

## Step 2: Plan Scope Analysis (Haiku agent)

Launch a **Haiku** agent with the plan text and summary from Step 1. The agent should:

1. List every file the plan proposes to **create**, **modify**, or **delete** (the file manifest)
2. For each file that already exists, read its current contents
3. Search the codebase for similar existing features or patterns (grep for related function names, class names, or module names mentioned in the plan)
4. Identify conventions used in those similar files (naming, structure, imports)

Return: the file manifest (create/modify/delete), contents of existing files, related files found, and detected conventions.

## Step 3: Checklist Validation (Sonnet agent)

Launch the **checklist-architect** agent. Pass it the outputs from Steps 1 and 2 as context.

Instruct the agent to return findings in this format:

```
ID: CHK-NNN
Severity: Critical | High | Medium | Low
Category: structure | completeness | quality | missing-phase
Description: <what the issue is>
Evidence: <specific checklist item or plan section reference>
Recommendation: <how to fix it>
Confidence: 0-100
```

The agent must also produce a complete proposed checklist organized by phase with dependency annotations.

## Step 4: Output

Present the checklist-architect's output directly:

---

### Checklist Review

**Findings** (if any issues were found)

List each CHK finding with its ID, severity, description, and recommendation.

---

**Proposed Implementation Checklist**

Include the complete checklist from checklist-architect, organized by phase with dependency annotations.

---
