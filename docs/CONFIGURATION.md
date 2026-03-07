# Spec Forge Configuration Guide

## Configuration File

Create `.spec-forge.json` in the project root to customize Spec Forge behavior.

## Default Directory Structure (Without Configuration)

```
project/
├── docs/                        # Default output: specification documents
│   └── <feature-name>/
│       ├── prd.md               # Product Requirements Document
│       ├── srs.md               # Software Requirements Specification
│       ├── tech-design.md       # Technical Design Document
│       └── test-plan.md         # Test Plan & Test Cases
│
├── ideas/                       # Default storage: brainstorming sessions
│   └── <idea-name>/
│       └── draft.md
```

## Complete Configuration Example

```json
{
  "version": "1.0",

  "_tool": {
    "name": "spec-forge",
    "description": "Professional software specification generator — from PRDs to Technical Design",
    "url": "https://github.com/tercel/spec-forge"
  },

  "directories": {
    "base": "",                    // Base directory (empty = project root)
    "output": "docs/",            // Output: specification documents
    "ideas": "ideas/",            // Storage: brainstorming sessions
    "features": "docs/features/"  // Output: lightweight feature specs
  },

  "templates": {
    "prd": "skills/prd-generation/references/template.md",
    "srs": "skills/srs-generation/references/template.md",
    "tech_design": "skills/tech-design-generation/references/template.md",
    "test_plan": "skills/test-plan-generation/references/template.md"
  },

  "git": {
    "auto_commit": false,          // Auto-commit after file generation
    "gitignore_patterns": [        // Auto-add to .gitignore
      "ideas/**"
    ]
  }
}
```

## Configuration Field Details

### directories

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `base` | string | `""` | Base directory (empty = project root) |
| `output` | string | `"docs/"` | Output path for formal specification documents |
| `ideas` | string | `"ideas/"` | Storage path for iterative brainstorming sessions |
| `features` | string | `"docs/features/"` | Output path for lightweight feature specs |

### templates

Allows overriding the default templates used for document generation. Useful for teams with their own standardized formats.

### git

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `auto_commit` | boolean | `false` | Auto-commit after file generation |
| `gitignore_patterns` | array | `["ideas/**"]` | Patterns to auto-add to .gitignore |

## Configuration File Location

### Priority (Highest to Lowest)

1. **Project Root** `.spec-forge.json`
2. **User Home Directory** `~/.spec-forge.json`
3. **Built-in Defaults**

---
*Created by [tercel](https://github.com/tercel). Optimized for Gemini CLI.*
