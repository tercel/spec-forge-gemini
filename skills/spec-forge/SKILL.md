---
name: spec-forge
description: Orchestrate full specification chain (PRD→SRS→Tech Design→Test Plan) or route subcommands.
instructions: >
  You are the spec-forge orchestrator. Your job is to route subcommands or run the full
  specification chain. Parse the feature name from arguments, determine the start stage,
  and execute the chain (PRD → SRS → Tech Design → Test Plan). If a subcommand is
  provided (e.g., /spec-forge prd), route to that specific skill.
---

# Spec Forge — Orchestrator

You are the spec-forge orchestrator. Your job is to route subcommands or run the full specification chain.

## Workflow

```
Input (Subcommand or Feature Name) → Parsing → Routing → Execution
```

## Parsing Arguments

Parse input into `subcommand` and `feature_name`:

| Input Pattern | subcommand | feature_name |
|---|---|---|
| `idea cool-feature` | `idea` | `cool-feature` |
| `feature cool-feature` | `feature` | `cool-feature` |
| `prd cool-feature` | `prd` | `cool-feature` |
| `srs cool-feature` | `srs` | `cool-feature` |
| `tech-design cool-feature` | `tech-design` | `cool-feature` |
| `test-plan cool-feature` | `test-plan` | `cool-feature` |
| `decompose cool-feature` | `decompose` | `cool-feature` |
| `cool-feature` (no known subcommand) | `chain` | `cool-feature` |
| (empty) | `dashboard` | — |

## Routing

### Route A: `dashboard` (no arguments)

Display spec-forge dashboard:

1. Scan `docs/` for existing spec documents (`docs/*/prd.md`, `docs/*/srs.md`, `docs/*/tech-design.md`, `docs/*/test-plan.md`)
2. Scan `docs/` for decomposed projects (`docs/project-*.md`)
3. Scan `docs/features/` for lightweight feature specs (`docs/features/*.md`)
4. Scan `ideas/` for active ideas
5. Display the dashboard summary.
6. Use `ask_user` tool to ask what to do next.

### Route B: `idea`

Invoke the `spec-forge:idea` skill. Pass `feature_name` as the argument.

### Route B2: `feature`

Invoke the `spec-forge:feature` skill. Pass `feature_name` as the argument.

### Route C: `prd` / `srs` / `tech-design` / `test-plan` (single document)

Invoke the corresponding skill:
- `prd` → invoke `spec-forge:prd-generation` skill with `feature_name`
- `srs` → invoke `spec-forge:srs-generation` skill with `feature_name`
- `tech-design` → invoke `spec-forge:tech-design-generation` skill with `feature_name`
- `test-plan` → invoke `spec-forge:test-plan-generation` skill with `feature_name`

### Route D: `chain` (full chain auto mode)

Run the full specification chain automatically for `feature_name`.

#### D.0: Scope Analysis

Launch `Task(subagent_type="general-purpose")`:
- Sub-agent prompt: "Invoke the spec-forge:decompose skill for '{feature_name}'."
- Wait for completion

Check result:
- If `docs/project-{feature_name}.md` exists → multi-split mode, go to D.0a
- Otherwise → single feature, proceed to D.1

#### D.0a: Multi-Split Chain Execution

Read `docs/project-{feature_name}.md` and parse the FEATURE_MANIFEST comment block to extract sub-feature names.

For each sub-feature in execution order:
Launch `Task(subagent_type="general-purpose")` to run the 4-document chain sequentially.

#### D.1: Detect Existing Progress

Scan `docs/` for existing documents matching `feature_name`. Determine which stages are already complete.

#### D.2: Check for Idea Draft

Check if `ideas/{feature_name}/` exists with status `ready` or `graduated`.

#### D.3: Execute Chain

Run each stage sequentially as a complete `Task` sub-agent.
1. **PRD** (requires user interaction)
2. **SRS** (chain mode)
3. **Tech Design** (chain mode)
4. **Test Plan** (chain mode)

#### D.4: Chain Completion

Display completion summary and suggest next steps.

### Route E: `decompose`

Launch `Task(subagent_type="general-purpose")` to invoke the `spec-forge:decompose` skill.
