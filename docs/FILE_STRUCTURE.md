# Spec Forge File Organization Specification

## Default Directory Structure

```
project-root/
├── docs/                            # Specification documents
│   ├── <feature-name>/              # Per-feature specification directory
│   │   ├── prd.md                   # Product Requirements Document
│   │   ├── srs.md                   # Software Requirements Specification
│   │   ├── tech-design.md           # Technical Design Document
│   │   └── test-plan.md             # Test Plan & Test Cases
│   │
│   └── features/                    # Lightweight feature specs (bridge to code-forge)
│       └── <feature-name>.md
│
├── ideas/                           # Brainstorming sessions (owned by spec-forge)
│   └── <idea-name>/                 # Per-idea directory
│       └── draft.md                 # Idea draft and validation status
│
├── .spec-forge.json                 # Spec Forge configuration (optional)
└── .gitignore
```

## Customizable Directory Structure

Customize directory locations through `.spec-forge.json` to avoid conflicts with existing projects.

See: [CONFIGURATION.md](./CONFIGURATION.md)

## Naming Conventions

### Feature and Idea directory naming
- Use kebab-case (lowercase + hyphens)
- Concise and descriptive
- Examples: `user-auth`, `payment-gateway`, `file-upload`

### Specification file naming
- Fixed names within feature directories: `prd.md`, `srs.md`, `tech-design.md`, `test-plan.md`
- This consistency enables automatic upstream/downstream document discovery.

### Idea draft naming
- Fixed name: `draft.md` within the idea directory.

## File Content Specification

### prd.md
Grounded in Google PRD, Amazon PR/FAQ, and Stripe Product Spec.
- Problem Statement
- User Personas & User Stories
- Features (P0/P1/P2)
- Success Metrics
- Mermaid User Journey

### srs.md
Grounded in IEEE 830 and ISO/IEC/IEEE 29148.
- Functional Requirements (FR-XXX-NNN)
- Non-functional Requirements (NFR-XXX-NNN)
- Data Model
- CRUD Matrix
- Traceability Matrix (PRD → SRS)

### tech-design.md
Grounded in Google Design Doc and RFC templates.
- Architecture Design (C4 Model Mermaid)
- Alternative Solution Comparison
- API & Data Design
- Security & Performance
- Parameter Validation Matrix

### test-plan.md
Grounded in IEEE 829 and ISTQB.
- Test Strategy (Pyramid)
- Test Case Specifications (TC-XXX-NNN)
- Data Integrity Tests (Real DB)
- Traceability Matrix (SRS → Test Cases)

---
*Created by [tercel](https://github.com/tercel). Optimized for Gemini CLI.*
