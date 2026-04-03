---
name: ai-traceability
description: Annotate every file you create or modify with a traceability header comment, and track all AI changes in ai-changelog.md. Use this skill to ensure full AI traceability across your project.
---

# AI Traceability Skill

Add a traceability header to every file you create or modify. Track all changes in `ai-changelog.md`.

A **session** = one AI interaction (chat thread, Codex task, agentic loop, or batch of completions). Each session gets **one** random 8-char hex ID. New prompt / new chat = new session.

## Header

```
AI Label:       ai-created | ai-evolved | ai-modified | ai-assisted
AI Changelog:   <8-char-hex>
AI Revisions:   <N>           ← +1 each session that touches the file
Risk Class:     low | medium | high  ← ratchets up, never down
AI Tool:        <tool-name>
AI Description: <what-changed> ← required except for ai-created
```

**Labels:** `ai-created` = fully AI-generated; `ai-evolved` = origin was `ai-created`, now AI-modified in a later session; `ai-modified` = human-created file modified by AI; `ai-assisted` = human-led with AI help.

**Escalation — within a session:** only escalate, never downgrade: `ai-created` → `ai-modified` → `ai-assisted` (same from `ai-evolved` → `ai-assisted`).

**Escalation — across sessions:** If origin label was `ai-created` → use `ai-evolved` in subsequent sessions. Otherwise → `ai-modified`. Human-led always → `ai-assisted`. Increment `AI Revisions` +1. `Risk Class` = max(current, all previous).

## Placement

Place **after** syntax-required first lines (shebangs, `<?php`, XML decls, `@charset`, `//go:build`), **after** pragmas (`"use strict"`, `"use client"`, `"use server"`, `# frozen_string_literal: true`, `# -*- coding: utf-8 -*-`, modelines), **after** license headers, **before** Python docstrings. Update in-place if header exists.

**Excluded:** JSON, binaries, CSV/TSV, lock files, auto-generated files — changelog only.

## Comment Syntax

| Style | Languages |
|-------|-----------|
| `/** … */` | PHP, JS/TS, Java, Kotlin, C/C++, C#, Dart |
| `# …` | Python, Ruby, Shell, YAML, Dockerfile, Make, HCL, Elixir, Erlang |
| `// …` | Go, Rust, Swift, Proto, GraphQL |
| `<!-- … -->` | HTML, XML |
| `/* … */` | CSS, SCSS |
| `-- …` | SQL, Lua |

Unlisted languages: use their standard comment syntax.

Example (Python):
```python
# AI Label: ai-evolved
# AI Changelog: e5f6g7h8
# AI Revisions: 3
# Risk Class: medium
# AI Tool: claude
# AI Description: Refactored parse_config for YAML support
```

## Risk

| Level | Scope |
|-------|-------|
| low | Docs, comments, config, styling, tests, formatting |
| medium | Business logic, features, refactors, dependencies |
| high | Auth, security, data migrations, API contracts, infra, PII |

Use highest applicable level when a change spans multiple categories.

## Changelog

Append to `ai-changelog.md` (project root; create with `# AI Changelog` header if missing):

```markdown
## <8-char-hex> — <YYYY-MM-DD HH:MM:SS UTC>
**Label:** <label>  **Risk:** <level>  **Tool:** <tool>
**Files:** `file1.py` (rev 2), `file2.ts` (rev 1)
**Lines:** `file1.py:12-45`, `file2.ts:8-20,55-60`  ← omit for ai-created
**Summary:** One-line description
```

The 8-char hex links the header to its changelog entry (must be unique within the changelog).

**Determining revision & label:** Search `ai-changelog.md` for the file. No prior entries → rev 1, label per within-session rules. Prior entries exist → rev = previous max + 1; check earliest entry's label to decide `ai-evolved` vs `ai-modified`; risk = max of current and all prior.

## Deletions, Renames & Moves

Record in changelog only (deleted files can't hold headers).

- **Delete:** `**Action:** deleted`, list removed file.
- **Rename/Move:** `**Action:** renamed` or `moved`, `` `old/path` → `new/path` `` in Files. Update destination header.
- **Copy + delete** (split): log creation + deletion in same entry.

```markdown
## f4e5d6c7 — 2026-04-03 12:00:00 UTC
**Label:** ai-modified  **Risk:** low  **Tool:** copilot
**Action:** renamed
**Files:** `utils/helpers.py` → `core/helpers.py` (rev 3)
**Summary:** Moved helpers into core package
```

## Human Overrides

If a human fully replaces an AI-annotated file's contents: remove the header, log `**Action:** human-override` in the changelog. History stays in changelog for audit. If AI later modifies the file, treat as fresh `ai-modified` (rev 1) — do not continue the old revision sequence.

## Team Merge Conflicts

To avoid conflicts on a shared `ai-changelog.md`, choose one strategy: (1) per-developer files (`ai-changelog-<user>.md`) merged by CI, (2) directory of session files (`ai-changelog/<hex>.md`), or (3) append-only at file bottom. The header `AI Changelog: <hex>` must always resolve to a findable entry.

## Formatter Compatibility

Auto-formatters may reposition headers — this is acceptable. If a formatter removes the annotation, move it to a preserved location (e.g., inside a docstring).
