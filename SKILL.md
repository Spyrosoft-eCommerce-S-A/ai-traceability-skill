---
name: ai-traceability
description: Annotate every file you create or modify with a traceability header comment, and track all AI changes in ai-changelog.md. Use this skill to ensure full AI traceability across your project.
---

# AI Traceability Skill

Annotate every file you create or modify with a traceability header. Track all changes in `ai-changelog.md`.

## Header Format

```
AI Label:       ai-created | ai-modified | ai-assisted
AI Changelog:   <8-char-hex>
Risk Class:     low | medium | high
AI Tool:        <tool-name>             ← e.g. copilot, codex, claude, gemini
AI Description: <what-changed>          ← required for ai-modified and ai-assisted; optional for ai-created
```

## Labels & Escalation

Labels only escalate within a session, never downgrade: `ai-created` > `ai-modified` > `ai-assisted`.

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
 * Risk Class: low
 * AI Tool: codex
 */
```

```python
# AI Label: ai-modified
# AI Changelog: e5f6g7h8
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
**Files:** `file1.py`, `file2.ts`
**Lines:** `file1.py:12-45`, `file2.ts:8-20,55-60`   ← for ai-modified and ai-assisted; omit for ai-created
**Summary:** One-line description
```

The 8-char hex is a random hexadecimal string unique within the changelog, linking the header annotation to its changelog entry.

## Formatter Compatibility

Auto-formatters may reposition headers — this is acceptable. If a formatter removes the annotation, move it to a preserved location (e.g., inside a docstring).
