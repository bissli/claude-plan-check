---
description: Validate a plan using 3 analysis agents (gaps, impact, risks)
---

# Plan Check: Review

Analyze an implementation plan for gaps, code impact, and risks (no checklist).

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

## Step 3: Parallel Validation (3 Sonnet agents)

Launch **all 3 agents in parallel**. Pass each agent the outputs from Steps 1 and 2 as context.

### Standard Issue Format

Instruct each agent to return findings in this format:

```
ID: PREFIX-NNN
Severity: Critical | High | Medium | Low
Category: <agent-specific category>
Description: <what the issue is>
Evidence: <file path, plan section, or rule reference>
Recommendation: <specific action to take>
Confidence: 0-100
```

### Agents to Launch

1. **gap-analyst** (GAP prefix): Analyze the plan for missing steps, edge cases, implicit assumptions, error handling gaps, backward compatibility issues, and knowledge coverage (check that the plan accounts for all `.claude/reference/` docs whose glob patterns match files the plan touches).

2. **code-impact-analyst** (IMP prefix): Trace dependencies of files the plan modifies, detect potential breakage, analyze side effects, and identify refactoring opportunities where the plan could reuse existing code.

3. **feasibility-risk-analyst** (RSK prefix): Assess technical feasibility, external dependencies, performance implications, security concerns. Produce a risk matrix (likelihood x impact x blast radius) for each risk.

## Step 4: Confidence Scoring + Deduplication (Haiku agents)

After all 3 agents complete:

1. Collect all findings from all agents
2. For any finding with **confidence < 80**, launch a parallel **Haiku** agent to re-evaluate it:
   - Give the agent the finding, the relevant plan section, and the relevant source code or rule
   - Ask it to return an updated confidence score (0-100) with a brief justification
3. **Deduplicate** across agents: if two findings describe the same underlying issue, keep the one with the highest confidence and note which agents flagged it
4. **Filter**: remove all findings with final confidence < 60

## Step 5: Synthesis and Output

Compile the final report in this structure:

---

### Plan Review Report

**Overall Assessment**: Pass | Pass with Warnings | Needs Revision

*(Pass = no Critical or High issues. Pass with Warnings = High issues but no Critical. Needs Revision = any Critical issues.)*

---

**Critical and High Issues** *(must address before implementation)*

List each issue with its ID, description, evidence, and recommendation. Group by severity.

---

**Medium and Low Issues** *(recommended to address)*

List each issue briefly. Group by agent.

---

**Refactoring Opportunities**

List any existing code the plan should leverage (from code-impact-analyst IMP findings with category=refactoring).

---

**Risk Assessment**

- Overall risk level: Low | Medium | High | Critical
- Blast radius: narrow | moderate | wide
- Top 3 risks (from feasibility-risk-analyst, by risk score)

---
