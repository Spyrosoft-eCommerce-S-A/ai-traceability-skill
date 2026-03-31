# AI Traceability Skill

A [Claude Code skill](https://code.claude.com/docs/en/skills) that ensures AI agents annotate every file they create or modify with a traceability header and track all changes in a centralized changelog.

## What It Does

- **Annotates files** with a structured header comment (label, changelog hash, risk class, tool, description)
- **Tracks changes** in a centralized `ai-changelog.md` with unique session hashes
- **Classifies risk** of each change (low / medium / high)
- **Adapts comment syntax** to the target language automatically
- **Handles edge cases** — shebangs, license headers, formats that don't support comments

## Labels

| Label | When |
|-------|------|
| `ai-created` | File fully created by AI |
| `ai-modified` | Existing file modified by AI |
| `ai-assisted` | Human-led, AI-assisted changes |

Labels only escalate within a session: `ai-created` → `ai-modified` → `ai-assisted`.

## Example

```php
<?php
/**
 * AI Label: ai-created
 * AI Changelog: a1b2c3d4
 * Risk Class: low
 * AI Tool: copilot
 */
```

## Install

Copy the skill into any of these locations:

| Scope | Path |
|-------|------|
| Personal (all projects) | `~/.claude/skills/ai-traceability/SKILL.md` |
| Project | `.claude/skills/ai-traceability/SKILL.md` |
| Additional directory | Use `--add-dir` to point Claude Code at this repo |

Claude will automatically apply the skill when creating or modifying files. You can also invoke it directly with `/ai-traceability`.
