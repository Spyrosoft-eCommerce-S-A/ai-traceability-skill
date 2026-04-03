# AI Traceability Skill

A [Claude Code skill](https://code.claude.com/docs/en/skills) that ensures AI agents annotate every file they create or modify with a traceability header and track all changes in a centralized changelog.

## What It Does

- **Annotates files** with a structured header comment (label, changelog hash, revision count, risk class, tool, description)
- **Tracks changes** in a centralized `ai-changelog.md` with unique session hashes
- **Tracks revisions** across sessions — revision counter increments each time a new session touches a file
- **Classifies risk** of each change (low / medium / high), ratcheting up across sessions
- **Adapts comment syntax** to the target language automatically
- **Handles edge cases** — shebangs, license headers, formats that don't support comments

## Labels

| Label | When |
|-------|------|
| `ai-created` | File fully created by AI in this session |
| `ai-evolved` | Originally `ai-created`, modified by AI in a subsequent session |
| `ai-modified` | Human-created file modified by AI |
| `ai-assisted` | Human-led, AI-assisted changes |

Within a session, labels escalate: `ai-created` → `ai-modified` → `ai-assisted`.  
Across sessions, an `ai-created` file becomes `ai-evolved` when modified again.

## Example

```php
<?php
/**
 * AI Label: ai-evolved
 * AI Changelog: a1b2c3d4
 * AI Revisions: 2
 * Risk Class: low
 * AI Tool: copilot
 * AI Description: Added input validation
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
