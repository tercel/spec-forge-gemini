---
name: srs
description: >
  Generates professional Software Requirements Specification (SRS) documents based on IEEE 830,
  ISO/IEC/IEEE 29148, and Amazon technical specification standards. This skill activates when the
  user needs a requirements document, requirements specification, SRS, functional requirements,
  non-functional requirements, software requirements, requirements analysis, or requirements
  engineering. It formalizes product needs into structured, testable, and traceable requirements
  with unique IDs, acceptance criteria, use cases, a CRUD matrix, and a full traceability matrix
  linking back to the upstream PRD.
instructions: >
  Generate a complete Software Requirements Specification following IEEE 830, ISO/IEC/IEEE 29148,
  and Amazon technical specification standards. Use the template at references/template.md and
  validate against references/checklist.md before finalizing.
---

# SRS Generation Skill

## What Is a Software Requirements Specification?

A Software Requirements Specification (SRS) is a formal document that describes exactly what a software system must do and the constraints under which it must operate. It serves as the contractual bridge between stakeholders who define the product vision (captured in a PRD) and the engineering team that designs, builds, and tests the system. A well-written SRS eliminates ambiguity, reduces rework, and provides a single source of truth for every requirement the system must satisfy.

The two foundational standards for SRS documents are **IEEE 830** (IEEE Recommended Practice for Software Requirements Specifications) and **ISO/IEC/IEEE 29148** (Systems and Software Engineering -- Life Cycle Processes -- Requirements Engineering). IEEE 830 established the canonical section structure -- introduction, overall description, specific requirements -- and defined the quality attributes every requirement must exhibit: correctness, unambiguity, completeness, consistency, ranking for importance, verifiability, modifiability, and traceability. ISO/IEC/IEEE 29148 modernized this foundation by integrating requirements engineering into the full systems and software lifecycle, emphasizing stakeholder needs analysis, requirements analysis, and requirements validation as continuous activities rather than one-time documentation events. This skill combines the structural rigor of both standards with the pragmatic, metric-driven approach found in Amazon technical specifications, where every requirement must be tied to a measurable outcome.

> **Note**: SRS is an on-demand document — it is NOT part of the default spec-forge auto chain (`idea → decompose → tech-design`). Use it when you need formal requirements traceability for compliance, audit, or complex system engineering.

## Generation Workflow

The SRS generation process follows a six-step workflow designed to produce a complete, high-quality document:

### Step 1: Scan Context

Before anything else, scan the current project to build context:

1. **Glob the project tree** (top 3 levels).
2. **Read the project README.md** if it exists.
3. **Scan the `docs/` directory** for existing documents.
4. **Detect (do NOT read)** matching PRD document: check if `docs/*/prd.md` related to the feature exists.
5. **Use Grep** to search for relevant code, APIs, and data models.

**Determine mode based on upstream discovery:**
- **Chain mode** (PRD found): Note the PRD file path. Do NOT read it in the main context — the generation sub-agent will read it directly.
- **Standalone mode** (no PRD found): Inform the user that no upstream PRD was found and extra questions will be asked.

### Step 2: Clarification Questions

**Chain mode**: Ask the user only 2-3 questions about areas the PRD does NOT cover (Functional Scope, Performance/Security specifics, etc.).

**Standalone mode**: Ask the user 5-8 key questions using `ask_user`:
- **Product Goal**: What is this feature trying to achieve?
- **Target Users**: Who are the primary users?
- **Feature Scope**: Main capabilities and out-of-scope items.
- **Performance/Security/Data/Integration**: Detailed technical requirements.
- **Success Criteria**: How will success be measured?

Wait for user responses before proceeding to Step 3.

### Step 3: Launch Document Generation

After receiving user answers, assemble and launch a generation sub-agent using `Task(subagent_type="general-purpose")`.

**Sub-agent Prompt Construction:**
- **Context**: Include project context summary, mode (Chain/Standalone), and user answers.
- **Upstream PRD** (Chain mode only): Provide the PRD file path and instruct the sub-agent to read it thoroughly.
- **Instructions**: Direct the sub-agent to read the generation instructions at `skills/srs-generation/references/generation-instructions.md`.
- **Anti-Shortcut Rules**: Explicitly mandate following the Anti-Shortcut Rules.
- **Output**: Direct writing to `docs/<feature-name>/srs.md`.

### Step 4: Generate the SRS (Sub-agent)

Using the template at `references/template.md`, the sub-agent generates the full SRS document. Every section of the IEEE 830 structure is populated. Requirements are assigned unique IDs (`FR-<MODULE>-<NNN>`, `NFR-<CATEGORY>-<NNN>`).

### Step 5: Traceability (Sub-agent)

If in Chain mode, the sub-agent builds a requirements traceability matrix (RTM) mapping PRD items to SRS requirements.

### Step 6: Quality Check & Next Steps

After the sub-agent returns:
1. Validate against `references/checklist.md`.
2. Present result summary, FR count, and NFR count to the user.
3. Suggest next steps:
   - Run `/spec-forge:tech-design` to design architecture.
   - Run `/spec-forge:test-plan` to plan testing.
   - Run `/code-forge:plan` to start implementation.

## Requirement ID Conventions

Every requirement receives a unique identifier that encodes its type and module or category:

- **Functional Requirements**: `FR-<MODULE>-<NNN>` where `<MODULE>` is a short uppercase label for the feature module (e.g., AUTH, CART, SEARCH, NOTIFY) and `<NNN>` is a zero-padded sequential number. Examples: FR-AUTH-001, FR-CART-012, FR-SEARCH-003.
- **Non-Functional Requirements**: `NFR-<CATEGORY>-<NNN>` where `<CATEGORY>` identifies the quality attribute (e.g., PERF, SEC, REL, AVL, MNT, PRT, USB) and `<NNN>` is a zero-padded sequential number. Examples: NFR-PERF-001, NFR-SEC-003, NFR-REL-002.

These IDs are used throughout the document -- in the requirements traceability matrix, in cross-references between related requirements, and in downstream documents such as technical designs and test plans. Consistent ID formatting is essential for automated traceability and search.

## Functional Requirements Writing Standards

Each functional requirement is structured as a complete use case specification with the following elements:

- **Requirement ID and Title**: The unique identifier and a concise descriptive title.
- **Description**: A clear statement of what the system shall do, written from the perspective of the system behavior rather than the implementation approach.
- **Actors**: The user classes or external systems that participate in this requirement.
- **Preconditions**: The conditions that must be true before the requirement can be exercised.
- **Main Flow**: A numbered sequence of steps describing the standard successful path through the use case.
- **Alternative Flows**: Branches from the main flow covering variations, error conditions, and edge cases.
- **Postconditions**: The observable state of the system after successful completion of the main flow.
- **Acceptance Criteria**: Specific, testable conditions that must be met for the requirement to be considered satisfied. Each acceptance criterion should be verifiable through inspection, demonstration, test, or analysis.
- **Priority**: The importance level of the requirement (P0 = must-have, P1 = should-have, P2 = nice-to-have), consistent with the prioritization used in the upstream PRD.
- **Source**: A reference back to the PRD item or stakeholder request that originated this requirement.

In addition to individual requirement specifications, the SRS includes a **CRUD matrix** -- a table that maps data entities (rows) against Create, Read, Update, and Delete operations (columns), with each cell indicating which functional requirement governs that operation. The CRUD matrix provides a rapid completeness check: if an entity has no "Delete" operation defined, that may be intentional (soft-delete policy) or an oversight that needs resolution.

## Non-Functional Requirements Categories

Non-functional requirements define the quality attributes and constraints of the system. Each NFR must include a specific, measurable metric, a target value, and a measurement method. The SRS organizes NFRs into the following categories:

- **Performance (NFR-PERF)**: Response times, throughput, latency percentiles (p50, p95, p99), concurrent user capacity, and resource utilization limits.
- **Security (NFR-SEC)**: Authentication mechanisms, authorization models, encryption standards, data protection measures, vulnerability scanning requirements, and compliance with security frameworks.
- **Reliability (NFR-REL)**: Mean time between failures (MTBF), mean time to recovery (MTTR), error rates, data integrity guarantees, and fault tolerance mechanisms.
- **Availability (NFR-AVL)**: Uptime SLAs (e.g., 99.9%), planned maintenance windows, disaster recovery objectives (RPO/RTO), and geographic redundancy requirements.
- **Maintainability (NFR-MNT)**: Code quality standards, documentation requirements, deployment frequency targets, and technical debt constraints.
- **Portability (NFR-PRT)**: Supported platforms, browsers, operating systems, container runtimes, and cloud provider compatibility.
- **Usability (NFR-USB)**: Accessibility standards (WCAG compliance level), internationalization requirements, maximum task-completion times, and user satisfaction targets.

## Requirements Traceability Matrix

The requirements traceability matrix (RTM) is a table that establishes bidirectional links between PRD items and SRS requirements. Each row maps a PRD identifier to one or more SRS functional or non-functional requirement IDs, along with a coverage status (Fully Covered, Partially Covered, or Not Covered). The RTM serves three purposes: it confirms that every product need has been addressed, it enables impact analysis when requirements change, and it provides the foundation for downstream traceability into technical design and test planning.

## Requirement Language Conventions

The SRS uses precise modal verbs to convey obligation levels, following IEEE 830 and RFC 2119 conventions:

- **"shall"**: Indicates a mandatory requirement. The system must satisfy this requirement to be considered compliant. Example: "The system shall authenticate users via OAuth 2.0 before granting access to protected resources."
- **"should"**: Indicates a recommended requirement. The system is expected to satisfy this requirement under normal circumstances, but justified exceptions are acceptable. Example: "The system should cache frequently accessed queries to reduce database load."
- **"may"**: Indicates an optional requirement. The system is permitted but not required to implement this behavior. Example: "The system may provide a dark-mode theme for the user interface."

Avoiding ambiguous language is critical. Terms like "fast," "user-friendly," "efficient," or "robust" are never used in isolation. Every qualitative claim must be paired with a quantitative target (e.g., "The system shall return search results within 200ms at the 95th percentile" rather than "The system shall be fast").

## Requirement Quality Attributes

Every requirement in the SRS -- functional or non-functional -- must satisfy four quality attributes:

1. **Unambiguous**: The requirement has exactly one interpretation. There is no room for disagreement about what the requirement means. Techniques for achieving unambiguity include using precise terminology defined in the glossary, avoiding pronouns with unclear antecedents, and stating explicit boundary conditions.
2. **Testable**: The requirement includes acceptance criteria or measurable targets that can be verified through testing, inspection, demonstration, or analysis. If a requirement cannot be tested, it must be rewritten until it can.
3. **Traceable**: The requirement can be traced both backward (to its source in the PRD or stakeholder request) and forward (to the design components and test cases that address it). Traceability is maintained through the requirement ID system and the RTM.
4. **Complete**: The requirement contains all information necessary for implementation. It specifies inputs, outputs, preconditions, postconditions, error handling, and boundary conditions without requiring the reader to make assumptions.

## Reference Files

The SRS generation skill relies on two reference files:

- **`references/template.md`**: The complete SRS document template following IEEE 830 structure. This template defines every section, provides placeholder guidance, and establishes the formatting conventions for requirements, tables, and diagrams.
- **`references/checklist.md`**: The quality checklist used during the final validation step. It contains items organized into four categories -- completeness, quality, consistency, and format -- that the generated document must satisfy before it is written to disk.

## Output Convention

The final SRS document is written to `docs/<feature-name>/srs.md` in the project root, where `<feature-name>` is a sanitized, lowercase, hyphen-separated slug derived from the user's input. The `docs/<feature-name>/` directory is created if it does not already exist. This naming convention places all documents for a feature in a single `docs/<feature-name>/` directory (`prd.md`, `srs.md`, `tech-design.md`, `test-plan.md`) and enables automatic upstream document discovery by downstream skills.

## Upstream Document Scanning

The skill automatically scans for upstream PRD documents before generation begins. It searches for files matching the `docs/*/prd.md` pattern, identifies the PRD most closely related to the requested feature, and reads it in full. The PRD content informs functional scope, user stories, priorities, success metrics, and scope boundaries. When the PRD is available, the SRS achieves significantly higher traceability and completeness. When it is not available, the skill relies on project context scanning and user clarification to fill the gap, and it notes in the generated document that PRD traceability is incomplete.
