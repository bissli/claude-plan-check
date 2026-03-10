# claude-plan-check

A Claude Code plugin that validates implementation plans using 4 parallel specialized agents.

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

Three subcommands are available:

```
/plan-check:all        # Full validation: 4 agents + checklist
/plan-check:review     # Analysis only: 3 agents (gaps, impact, risks)
/plan-check:checklist  # Checklist only: evaluate and generate checklist
```

Reads the most recent plan from `~/.claude/plans/`. If no plan file is found, falls back to extracting plan text from conversation context.

## What It Does

The `/plan-check:all` command runs a 5-step validation pipeline:

1. **Context gathering** (Haiku) -- reads project rules, CLAUDE.md files, and the plan
2. **Scope analysis** (Haiku) -- builds a file manifest and finds related patterns
3. **Parallel validation** (4 Sonnet agents) -- deep analysis across 4 dimensions
4. **Confidence scoring** (Haiku) -- re-evaluates low-confidence findings, deduplicates
5. **Synthesis** -- produces a structured validation report

`/plan-check:review` runs steps 1-2 with 3 analysis agents (no checklist). `/plan-check:checklist` runs steps 1-2 with the checklist agent only.

## Agents

| Agent                        | Prefix | Focus                                                                                                |
| ---------------------------- | ------ | ---------------------------------------------------------------------------------------------------- |
| **gap-analyst**              | GAP    | Missing steps, edge cases, implicit assumptions, error handling, backward compat, knowledge coverage |
| **code-impact-analyst**      | IMP    | Dependency breakage, side effects, refactoring opportunities                                         |
| **feasibility-risk-analyst** | RSK    | Technical feasibility, performance, security, risk matrix                                            |
| **checklist-architect**      | CHK    | Checklist structure, completeness, item quality, phase ordering                                      |

## Output

The full report (`/plan-check:all`) includes:

- **Overall Assessment** -- Pass / Pass with Warnings / Needs Revision
- **Critical and High Issues** -- must address before implementation
- **Medium and Low Issues** -- recommended improvements
- **Refactoring Opportunities** -- existing code the plan should leverage
- **Risk Assessment** -- overall risk level, blast radius, top 3 risks
- **Implementation Checklist** -- validated/improved, phased with dependencies

## Confidence Threshold

Findings below 60% confidence are filtered out (lower than code-review's 80% threshold because fixing a plan is cheap -- surface more marginal findings).

## License

MIT
