---
name: spec-forge
description: Orchestrate full specification chain (Idea→Decompose→Tech Design + Feature Specs) or route subcommands.
instructions: >
  You are the spec-forge orchestrator. Your job is to route subcommands or run the full
  specification chain (Idea → Decompose → Tech Design + Feature Specs). Parse the feature
  name from arguments, determine the start stage, and execute the chain. If a subcommand
  is provided (e.g., /spec-forge prd), route to that specific skill. PRD, SRS, and Test Plan
  are on-demand and NOT part of the auto chain.
---

# Spec Forge — Orchestrator

You are the spec-forge orchestrator. Your job is to route subcommands or run the full specification chain.

## Workflow

```
Input (Subcommand or Feature Name) → Parsing → Routing → Execution
```

### Step 1: Parse Arguments

Parse arguments into `subcommand` and `feature_name`:

| Input Pattern | subcommand | feature_name |
|---|---|---|
| `idea cool-feature` | `idea` | `cool-feature` |
| `prd cool-feature` | `prd` | `cool-feature` |
| `srs cool-feature` | `srs` | `cool-feature` |
| `tech-design cool-feature` | `tech-design` | `cool-feature` |
| `test-plan cool-feature` | `test-plan` | `cool-feature` |
| `decompose cool-feature` | `decompose` | `cool-feature` |
| `cool-feature` (no known subcommand) | `chain` | `cool-feature` |
| (empty) | `dashboard` | — |

### Step 2: Route

#### Route A: `dashboard` (no arguments)

Display spec-forge dashboard:

1. Scan `docs/` for existing spec documents (`docs/*/prd.md`, `docs/*/srs.md`, `docs/*/tech-design.md`, `docs/*/test-plan.md`)
2. Scan `docs/` for decomposed projects (`docs/project-*.md`)
3. Scan `docs/features/` for lightweight feature specs (`docs/features/*.md`)
4. Scan `ideas/` for active ideas
5. Display:

```
spec-forge — Professional Software Specification Generator

Active Ideas (in ideas/):
  # | Idea            | Status     | Sessions | Last Updated
  1 | cool-feature    | refining   | 3        | 2026-02-10
  2 | another-idea    | exploring  | 1        | 2026-02-14

Projects (in docs/):
  # | Project         | Sub-Features | Manifest
  1 | my-project      | 3            | docs/project-my-project.md

Feature Specs (in docs/features/):
  # | Feature         | Source Tech Design
  1 | core-executor   | docs/my-feature/tech-design.md
  2 | schema-system   | docs/my-feature/tech-design.md

Specifications (in docs/):
  Feature        | PRD | SRS | Tech Design | Test Plan
  user-login     |  +  |  +  |      +      |     +
  payment        |  +  |  +  |             |

Commands:
  /spec-forge:idea <name>          Start or resume brainstorming
  /spec-forge:decompose <name>     Decompose project into sub-features
  /spec-forge <name>               Run full chain (Idea → Decompose → Tech Design + Feature Specs)
  /spec-forge:tech-design <name>   Generate Tech Design + Feature Specs
  /spec-forge:prd <name>           Generate PRD (on-demand, for stakeholders)
  /spec-forge:srs <name>           Generate SRS (on-demand, for compliance)
  /spec-forge:test-plan <name>     Generate Test Plan (on-demand, for QA)
```

6. Use `ask_user` to ask what to do next.

#### Route B: `idea`

Invoke the `spec-forge:idea` skill. Pass `feature_name` as the argument.

#### Route C: `prd` / `srs` / `tech-design` / `test-plan` (single document)

Invoke the corresponding skill:
- `prd` → invoke `spec-forge:prd` skill with `feature_name`
- `srs` → invoke `spec-forge:srs` skill with `feature_name`
- `tech-design` → invoke `spec-forge:tech-design` skill with `feature_name`
- `test-plan` → invoke `spec-forge:test-plan` skill with `feature_name`

#### Route D: `chain` (full chain auto mode)

Run the full specification chain automatically for `feature_name`. The chain consists of three stages:

1. **Idea** — Validate requirements and crystallize the concept (interactive)
2. **Decompose** — Determine if the project needs splitting into sub-features
3. **Tech Design** — Generate architecture document + auto-generate feature specs in `docs/features/`

> **Note**: PRD, SRS, and Test Plan are NOT part of the auto chain. They can be generated on-demand via `/spec-forge:prd`, `/spec-forge:srs`, or `/spec-forge:test-plan` when needed (e.g., for stakeholder alignment, compliance, or formal QA). The tech-design in standalone mode captures requirements directly through targeted questions, eliminating the need for intermediate PRD/SRS documents.

##### D.0: Check for Existing Idea

Check if `ideas/{feature_name}/` exists:
- If status is `ready` or `graduated`: note the idea is available as context, proceed to D.1
- If status is `exploring` or `refining`: warn user that the idea is still in progress, suggest running `/spec-forge:idea {feature_name}` to finish it first
- If no idea exists: proceed to D.0a (start with idea stage)

##### D.0a: Stage 1 — Idea

Launch `Task(subagent_type="general-purpose")`:
- Sub-agent prompt: "Invoke the spec-forge:idea skill for '{feature_name}'. Guide the user through iterative discovery to validate the concept and produce a draft at ideas/{feature_name}/draft.md."
- Wait for completion

After sub-agent returns, check `ideas/{feature_name}/state.json`:
- If `status` is `ready` or `graduated`: proceed to D.1
- If `status` is `parked`: inform user — *"The idea was parked. Run `/spec-forge:idea {feature_name}` to resume when ready."* — stop chain
- If `status` is `exploring`, `researching`, or `refining`: ask user via `ask_user`:
  - **Continue anyway** — proceed to D.1 using partial draft as context
  - **Finish idea first** — stop chain, user resumes with `/spec-forge:idea {feature_name}`

##### D.1: Stage 2 — Decompose

Launch `Task(subagent_type="general-purpose")`:
- Sub-agent prompt: "Invoke the spec-forge:decompose skill for '{feature_name}'."
- Wait for completion

Check result:
- If `docs/project-{feature_name}.md` exists → multi-split mode, go to D.1a
- Otherwise → single feature, proceed to D.2

##### D.1a: Multi-Split Chain Execution

Read `docs/project-{feature_name}.md` and parse the FEATURE_MANIFEST block.

For each sub-feature in execution order, launch `Task(subagent_type="general-purpose")`:
- Sub-agent prompt: "Invoke the spec-forge:tech-design skill for '{sub_feature_name}'. Idea-first mode. Read ideas/{feature_name}/draft.md for overall context and docs/project-{feature_name}.md for specific scope. IMPORTANT: This will also auto-generate feature specs in docs/features/."
- Wait for completion

##### D.2: Stage 3 — Tech Design

Before launching, check whether `ideas/{feature_name}/draft.md` exists and build the sub-agent prompt accordingly:

- **If idea draft exists**: prompt = "Invoke the spec-forge:tech-design skill for '{feature_name}'. Idea-first mode. Read ideas/{feature_name}/draft.md for requirements context. IMPORTANT: After writing the tech-design, also auto-generate feature specs in docs/features/."
- **If no idea draft**: prompt = "Invoke the spec-forge:tech-design skill for '{feature_name}'. Standalone mode — no upstream idea draft found. Ask the user the full set of standalone clarification questions. IMPORTANT: After writing the tech-design, also auto-generate feature specs in docs/features/."

Launch `Task(subagent_type="general-purpose")` with the appropriate prompt above.
Wait for completion.

##### D.3: Chain Completion

Display final result:
```
spec-forge chain complete: {feature_name}
  [x] Idea           ideas/{feature_name}/
  [x] Decompose      ...
  [x] Tech Design    docs/{feature_name}/tech-design.md
  [x] Feature Specs  docs/features/overview.md + {count} specs
```

#### Route E: `decompose`

Launch `Task(subagent_type="general-purpose")`:
- Sub-agent prompt: "Invoke the spec-forge:decompose skill for '{feature_name}'."
- Wait for completion
