# claude-plan-check

A Claude Code plugin that validates implementation plans using parallel specialized agents and **edits the plan directly** to address all findings.

## Installation

Add the marketplace and install:

```
/plugin marketplace add bissli/claude-plan-check
/plugin install plan-check@bissli-claude-plan-check
```

For local development:

```
claude --plugin-dir ./claude-plan-check
```

## Usage

Four subcommands are available:

```
/plan-check:verify     # Verify correctness, completeness, assumptions
/plan-check:gap        # Analyze what existing code might break
/plan-check:checklist  # Add or improve an implementation checklist
/plan-check:deep       # Deep analysis: all 4 agents in parallel
```

Reads the most recent plan from `~/.claude/plans/`. If no plan file is found, falls back to extracting plan text from conversation context. When the plan comes from conversation context, the amended plan is written to `~/.claude/plans/amended-plan.md`.

## How It Works

Every command follows the same pattern:

1. **Context gathering** (Haiku) -- reads project rules, CLAUDE.md files, and locates the plan file
2. **Scope analysis** (Haiku) -- builds a file manifest, reads existing files, maps callers
3. **Agent analysis** (Sonnet) -- specialized agents analyze the plan and return findings with plan amendments
4. **Update plan** -- the orchestrating command applies all amendments directly to the plan file

The updated plan IS the deliverable. No standalone report is produced.

### Commands

**`/plan-check:verify`** launches the plan-verifier agent for correctness, completeness, edge cases, error handling, assumptions, test quality, and knowledge coverage.

**`/plan-check:gap`** launches the breakage-analyst agent to trace callers, detect interface changes, import cascades, shared state issues, and recommend regression tests.

**`/plan-check:checklist`** launches the checklist-architect agent to evaluate or create an implementation checklist, then inserts it into the plan.

**`/plan-check:deep`** is the most thorough analysis. It launches all 4 agents in parallel (plan-verifier, breakage-analyst, test-reviewer, simplification-analyst), deduplicates findings, then runs a second wave of Haiku agents to re-evaluate Critical and High findings. All confirmed amendments are applied to the plan.

## Agents

| Agent                      | Prefix | Color   | Focus                                                              |
| -------------------------- | ------ | ------- | ------------------------------------------------------------------ |
| **plan-verifier**          | VFY    | yellow  | Correctness, completeness, edge cases, assumptions, test quality   |
| **breakage-analyst**       | BRK    | red     | Caller breakage, interface changes, import cascades, test breakage |
| **test-reviewer**          | TST    | cyan    | Test coverage, proposed test quality, missing scenarios, smells    |
| **simplification-analyst** | SMP    | magenta | Code reuse, over-engineering, pattern conformance, consolidation   |
| **checklist-architect**    | CHK    | green   | Checklist structure, completeness, item quality, phase ordering    |

## Plan Amendment Model

Agents are read-only -- they analyze the codebase and return structured findings with plan amendments. The orchestrating command (the main Claude thread) collects amendments and edits the plan file.

Each amendment specifies an operation (`add`, `replace`, `remove`, or `append-section`), a target location in the plan, and the content to apply. Amendments are processed in severity order (Critical first). Conflicts between amendments targeting the same section are resolved by applying the higher-severity amendment.

## Confidence Threshold

In `/plan-check:deep`, the second wave of Haiku agents re-evaluates Critical and High findings. Findings below 60% confidence are filtered out after the second wave.

## Migration from v1

| v1 Command              | v2 Equivalent                            |
| ----------------------- | ---------------------------------------- |
| `/plan-check:all`       | `/plan-check:deep`                       |
| `/plan-check:review`    | `/plan-check:verify` + `/plan-check:gap` |
| `/plan-check:checklist` | `/plan-check:checklist` (rewritten)      |

Key differences in v2:
- Commands edit the plan file directly instead of producing standalone reports
- All severity levels are addressed (Low through Critical), not just reported
- New agents: plan-verifier (VFY), breakage-analyst (BRK), test-reviewer (TST), simplification-analyst (SMP)
- Removed agents: gap-analyst (GAP), code-impact-analyst (IMP), feasibility-risk-analyst (RSK)

## License

MIT
