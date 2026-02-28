# Spec Forge Troubleshooting Guide

## Configuration-Related Issues

### Q1: Configuration file not taking effect
**Symptoms**: Created `.spec-forge.json` but Spec Forge is still using default configuration.
**Solutions**:
1. Confirm the file is in the project root directory.
2. Validate JSON format: `cat .spec-forge.json | jq .`.
3. Check the filename: must be `.spec-forge.json`, not `.forge.json` or `spec-forge.json`.

---

## Document Generation Issues

### Q2: Sub-agent fails to generate document
**Symptoms**: The sub-agent starts but returns an error or empty result.
**Solutions**:
1. **Check context**: Did you provide enough information in the Step 2 clarification questions?
2. **Check templates**: Ensure the template files in `skills/*/references/template.md` exist and are readable.
3. **Task length**: If the feature is extremely large, the sub-agent might hit a token limit. Try decomposing the project first with `/spec-forge:decompose`.

### Q3: Upstream document not detected
**Symptoms**: Running `/spec-forge:srs` but it says "Standalone mode" even though a PRD exists.
**Solutions**:
1. Ensure the PRD is at the expected path: `docs/<feature-name>/prd.md`.
2. Ensure the `<feature-name>` in the path matches the argument you passed to the command.
3. Check for hidden characters or spelling differences in the directory names.

---

## Git and Workspace Issues

### Q4: Accidentally committed brainstorming drafts
**Symptoms**: `ideas/` directory is cluttering your Git history.
**Solutions**:
1. Add `ideas/` to your `.gitignore`.
2. Remove from Git (keep local): `git rm -r --cached ideas/`.
3. Commit the change: `git commit -m "chore: ignore brainstorming ideas"`.

---

## Common Error Messages

| Error Message | Cause | Solution |
|---------|------|---------|
| `Template not found` | Custom template path in `.spec-forge.json` is wrong | Verify the path is relative to project root |
| `Permission denied` | Cannot write to `docs/` or `ideas/` | Check folder permissions |
| `Invalid sub-agent prompt` | Malformed instructions in `SKILL.md` | Check the latest updates to the skill |

---
*Created by [tercel](https://github.com/tercel). Optimized for Gemini CLI.*
