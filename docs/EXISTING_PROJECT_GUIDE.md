# Using Spec Forge in Existing Projects

> How to use Spec Forge in projects that are already in development or completed.

## Applicable Scenarios
- ✅ Project is already in progress and needs formal requirements.
- ✅ Some documents exist (e.g., a README or an old PRD) but need standardization.
- ✅ Project is completed and needs "as-built" documentation for maintenance or team onboarding.
- ✅ You want to establish traceability between existing code and new requirements.

---

## Usage Methods

### Method 1: Forward-looking (Standard)
Use this for new features being added to an existing project.
1. Run `/spec-forge:prd <feature-name>` to start the chain.
2. Spec Forge will scan your existing project to ensure the new feature aligns with current architecture and conventions.

### Method 2: Retrospective (Documentation-only)
Use this to document features that are already implemented.
1. Create a lightweight feature description or point Spec Forge to existing code.
2. Run the spec chain: `/spec-forge:prd`, `/spec-forge:srs`, etc.
3. **Sub-agent Prompt Hint**: Tell the sub-agent "This feature is already implemented in `src/auth/`. Read the code and generate the PRD/SRS to reflect the current implementation while identifying any gaps or technical debt."

### Method 3: Hybrid (Refinement)
Use this when you have some notes or a draft but need a professional spec.
1. Place your notes in `ideas/<feature-name>/draft.md`.
2. Run `/spec-forge <feature-name>`.
3. Spec Forge will use your draft as primary context and formalize it into the full 4-document chain.

---

## Smart Upstream Detection

Spec Forge is designed to be "chain-aware." When you run a command like `/spec-forge:tech-design`, it will:
1. **Scan `docs/`**: Look for `prd.md` and `srs.md` for that feature.
2. **If found**: Run in **Chain Mode**. It will extract requirements and goals directly from those docs, asking you only 3-5 technical clarification questions.
3. **If NOT found**: Run in **Standalone Mode**. It will ask you 8-12 comprehensive questions to establish the missing product and requirements context before designing.

This makes it easy to jump into the specification process at any stage.

---

## Best Practices for Existing Projects

### ✅ Scan Before You Write
Always let Spec Forge perform its Step 1 (Project Context Scanning). This ensures the generated documents use your project's terminology, tech stack, and architectural patterns.

### ✅ Document Technical Debt
When documenting existing features, use the "Risks" or "Constraints" sections to record known technical debt or areas that need refactoring.

### ✅ Use as a Refactoring Guide
If you're planning a major refactor, generate the "Ideal" Tech Design first, then use it as the target state for your implementation tasks.

---
*Created by [tercel](https://github.com/tercel). Optimized for Gemini CLI.*
