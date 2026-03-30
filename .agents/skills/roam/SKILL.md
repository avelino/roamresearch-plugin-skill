---
name: roam
description: Guidelines for working with Roam Research graphs via MCP server — batch actions, block/page operations, Roam-flavored Markdown syntax, daily page date format, and common pitfalls. Use when operating on Roam graphs, creating/updating blocks, searching pages, or any roam_* MCP tool interaction.
license: MIT
metadata:
  version: 1.0.0
---

# Roam Research Skill

Use this skill for any task that involves reading, writing, or organizing content in a Roam Research graph via the MCP server.

## Triggers

- User asks to capture, log, or append notes to Roam
- User references "daily page", "[[page]]", "((uid))", "block", "graph", or "roam"
- User wants to search, retrieve, or cross-reference knowledge in Roam
- User asks to create a new page or outline structure in Roam
- Any `roam_*` MCP tool appears in context

## Critical Rules

1. **Load the cheatsheet first.** Before any write operation, call `roam_get_resource` with the Roam Markdown Cheatsheet URI to load correct syntax into context.
2. **Prefer UIDs over titles.** Use `roam_fetch_page_by_title` to get UIDs; then operate by UID to avoid case-sensitivity bugs and ambiguity.
3. **Batch everything.** Use `roam_process_batch_actions` for all create/update/move/delete operations — it is the canonical write tool since v1.36.0. Individual tools like `roam_create_block` and `roam_update_block` are **deprecated**.
4. **Block references use `((uid))`, page links use `[[page title]]`.** Never swap them. `((uid))` is always a 9-char alphanumeric string, never a page title.
5. **Daily page title format:** `Month Dth, YYYY` with ordinal suffix — e.g., `March 30th, 2026`. **Never** use ISO format (`2026-03-30`).
6. **Tags:** `#tag` for single word, `#[[multi word tag]]` for multi-word. No spaces without double brackets.
7. **Nested bullets = nested `children` in batch actions**, not indented markdown. Roam's hierarchy is positional, not textual.
8. **`roam_search_by_text` is case-sensitive.** If search returns empty, retry with lowercase or a shorter substring.
9. **Never assume a page exists.** Always fetch first with `roam_fetch_page_by_title`; create only if it returns null.
10. **Write-protected graphs require `write_key`.** If the operation fails with a permissions error, surface this to the user — do not retry silently.
11. **Block content is plain text + Roam-flavored Markdown.** HTML is not supported.
12. **`roam_remember` / `roam_recall` use a configurable tag** (`ROAM_MEMORIES_TAG`). Do not hardcode — check graph config or ask the user.
13. **Multi-graph setups:** pass `graph` parameter explicitly if more than one graph is configured. Default is `ROAM_DEFAULT_GRAPH`.
14. **Block order matters.** In `roam_process_batch_actions`, actions execute top-to-bottom. Build parent blocks before children.
15. **Resolving references:** `roam_fetch_page_by_title` with `resolve_refs: true` resolves transclusions up to 4 levels deep.

## Common Mistakes

| Mistake | Correct |
|---------|---------|
| `2026-03-30` for daily page | `March 30th, 2026` |
| `roam_create_block` / `roam_update_block` | `roam_process_batch_actions` |
| `((My Page Title))` | `((9charUID))` for block refs, `[[My Page Title]]` for page links |
| Writing without loading cheatsheet | Call `roam_get_resource` first |
| Creating page without checking if it exists | `roam_fetch_page_by_title` first |
| `#multi word tag` | `#[[multi word tag]]` |

## Key Tools

| Tool | Purpose |
|------|---------|
| `roam_fetch_page_by_title` | Read page content + get UIDs |
| `roam_process_batch_actions` | All writes: create/update/move/delete blocks |
| `roam_search_by_text` | Full-text search (case-sensitive) |
| `roam_search_for_tag` | Find all blocks with a given tag |
| `roam_create_page` | Create a new page (batch_actions can also do this) |
| `roam_remember` / `roam_recall` | Persistent memory via tagged blocks |
| `roam_get_resource` | Load cheatsheet and other resources |

## Quick Syntax Reference

```
[[Page Link]]           links to page
((block-uid))           transcludes a block
#tag / #[[multi word]]  tags
^^highlighted^^         highlight
**bold** / __italic__   formatting
{{[[TODO]]}}            checkbox (unchecked)
{{[[DONE]]}}            checkbox (checked)
```

## Common Patterns

### Append to today's daily page
```
1. roam_fetch_page_by_title(title="March 30th, 2026") -> get page UID
2. roam_process_batch_actions([
     { action: "create-block", location: { page: <uid>, order: "last" },
       block: { string: "content" } }
   ])
```

### Create structured outline
```
roam_process_batch_actions([
  { action: "create-block", location: { page: <uid>, order: "last" },
    block: { string: "## Project Alpha", uid: "parent-uid" } },
  { action: "create-block", location: { block: "parent-uid", order: "last" },
    block: { string: "Goals" } }
])
```

### Search then update
```
1. roam_search_by_text(query="keyword") -> get block UIDs
2. roam_process_batch_actions([
     { action: "update-block", block: { uid: <uid>, string: "updated content" } }
   ])
```

## Learning Loop

The agent saves learnings to `[[ai-agent/learning]]` in the user's graph. This builds a persistent, curated knowledge base that survives across sessions and is reviewable by the user.

### Session start

At the beginning of every session that involves Roam operations, load prior learnings:

```
roam_fetch_page_by_title(title="ai-agent/learning") -> read all blocks
```

Use these learnings as context for all subsequent operations. They override generic rules in this skill when there's a conflict (user preferences > defaults).

### When to save

- **Failed operation** — after fixing a Roam error, save what went wrong and the fix
- **Graph conventions discovered** — page naming patterns, tag taxonomy, recurring templates, block structure
- **User corrections** — if the user corrects your output (format, structure, tag choice), save the preference immediately
- **New patterns** — how the user organizes daily pages, projects, meetings, etc.

### When NOT to save

- Generic Roam syntax (already in the skill)
- One-off facts that won't be useful later
- Content the user explicitly asked to keep private
- Duplicates — check existing blocks on `[[ai-agent/learning]]` before saving

### How to save

Append a block to `[[ai-agent/learning]]` with the date and category:

```
1. roam_fetch_page_by_title(title="ai-agent/learning") -> get page UID
   (if null, create the page first with roam_create_page)
2. roam_process_batch_actions([
     { action: "create-block",
       location: { page: <uid>, order: "last" },
       block: { string: "[[March 30th, 2026]] #[[convention]] daily pages always start with #[[standup]] followed by #[[tasks]]" } }
   ])
```

### Block format

```
[[<date>]] #[[<category>]] <learning>
```

Categories: `#[[convention]]`, `#[[fix]]`, `#[[preference]]`, `#[[pattern]]`

This creates a filterable log — the user can query by category or date using Roam's native queries.

## Reference Files

- [`reference/batch-actions.md`](reference/batch-actions.md) — complete `roam_process_batch_actions` action schema
- [`reference/roam-markdown.md`](reference/roam-markdown.md) — full Roam-flavored Markdown syntax + daily page date format
