---
name: ai-traceability
description: Annotate every file you create or modify with a traceability header comment, and track all AI changes in ai-changelog.md. Use this skill to ensure full AI traceability across your project.
---

# AI Traceability Skill

Annotate every file you create or modify with a traceability header. Track all changes in `ai-changelog.md`.

## Header Format

```
AI Label:       ai-created | ai-evolved | ai-modified | ai-assisted
AI Changelog:   <8-char-hex>
AI Revisions:   <N>                     ← increments each session that touches the file
Risk Class:     low | medium | high     ← highest ever seen across all sessions (ratchet up, never down)
AI Tool:        <tool-name>             ← current session's tool (e.g. copilot, codex, claude, gemini)
AI Description: <what-changed>          ← required for ai-modified, ai-evolved, and ai-assisted; optional for ai-created
```

## Labels & Escalation

| Label | When |
|-------|------|
| `ai-created` | File fully created by AI in this session |
| `ai-evolved` | File was originally `ai-created`, now modified by AI in a subsequent session |
| `ai-modified` | Human-created file modified by AI |
| `ai-assisted` | Human-led, AI-assisted changes |

### Within a session

Labels only escalate, never downgrade: `ai-created` → `ai-modified` → `ai-assisted`.
Same rule applies when starting from `ai-evolved`: `ai-evolved` → `ai-assisted`.

### Across sessions

- If the file's **origin label** (from its first changelog entry) was `ai-created`, use `ai-evolved` instead of `ai-modified` in all subsequent sessions.
- If the origin was `ai-modified` or `ai-assisted`, keep using `ai-modified` in subsequent sessions.
- When a new session touches a file, increment `AI Revisions` by 1.
- `Risk Class` ratchets to the **highest ever seen** across all sessions — never downgrade.

## Placement

1. Place **after** any syntax-required first lines (shebangs, `<?php`, XML declarations, `@charset`, encoding declarations, `//go:build`).
2. Place **after** any existing license header.
3. **Update in-place** if the file already has a traceability header.

## Excluded Files

Do **not** annotate: JSON, binary files, CSV/TSV, lock files, auto-generated files. Record these changes only in `ai-changelog.md`.

## Comment Syntax

Use the language's standard comment style:

| Style | Languages |
|-------|-----------|
| `/** ... */` | PHP (`<?php`), JS/TS, Java, Kotlin, C/C++, C#, Dart |
| `# ...` | Python, Ruby, Shell, YAML, Dockerfile, Make, Terraform/HCL, Elixir, Erlang |
| `// ...` | Go, Rust, Swift, Proto, GraphQL |
| `<!-- ... -->` | HTML, XML |
| `/* ... */` | CSS, SCSS |
| `-- ...` | SQL, Lua |

For unlisted languages, use their standard comment syntax.

### Examples

```php
<?php
/**
 * AI Label: ai-created
 * AI Changelog: a1b2c3d4
 * AI Revisions: 1
 * Risk Class: low
 * AI Tool: codex
 */
```

```python
# AI Label: ai-evolved
# AI Changelog: e5f6g7h8
# AI Revisions: 3
# Risk Class: medium
# AI Tool: claude
# AI Description: Refactored parse_config for YAML support
```

## Risk Classification

| Risk | Scope |
|------|-------|
| low | Docs, comments, config, styling, tests, formatting |
| medium | Business logic, features, refactors, dependencies |
| high | Auth, security, data migrations, API contracts, infra, PII |

Use the **highest** applicable level when a change spans multiple risks.

## Changelog

Add an entry to `ai-changelog.md` (project root) after completing work. Insert at any position (top recommended). Create the file with header `# AI Changelog` if missing.

```markdown
## <8-char-hex> — <YYYY-MM-DD HH:MM:SS UTC>
**Label:** <label>
**Risk:** <low|medium|high>
**Tool:** <tool-name>
**Revision:** <N>                                     ← file-level revision number for each touched file
**Files:** `file1.py`, `file2.ts`
**Lines:** `file1.py:12-45`, `file2.ts:8-20,55-60`   ← for ai-modified, ai-evolved, and ai-assisted; omit for ai-created
**Summary:** One-line description
```

The 8-char hex is a random hexadecimal string unique within the changelog, linking the header annotation to its changelog entry.

### Determining revision and label for a file

1. Search `ai-changelog.md` for prior entries that list the file.
2. If none exist → `AI Revisions: 1`. Label follows within-session rules.
3. If entries exist → `AI Revisions: <previous max + 1>`. Check the **earliest** entry's label:
   - If it was `ai-created` → use `ai-evolved` (unless human-led, then `ai-assisted`).
   - Otherwise → use `ai-modified` (unless human-led, then `ai-assisted`).
4. Set `Risk Class` to the **highest** between the current change and any previous entry for the file.

## Formatter Compatibility

Auto-formatters may reposition headers — this is acceptable. If a formatter removes the annotation, move it to a preserved location (e.g., inside a docstring).
