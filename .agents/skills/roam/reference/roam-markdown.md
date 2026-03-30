# Roam-Flavored Markdown

## Links & References

```
[[Page Title]]           bidirectional page link
((9charUID))             block transclusion (renders block content inline)
[alias]([[Page Title]])  aliased page link
[alias]((uid))           aliased block reference
```

## Tags

```
#tag                     single-word tag (no spaces)
#[[multi word tag]]      multi-word tag
```

## Text Formatting

```
**bold**
__italic__
^^highlighted^^
~~strikethrough~~
`inline code`
```

## Block Types

```
{{[[TODO]]}} task text   unchecked checkbox
{{[[DONE]]}} task text   checked checkbox
{{[[query]]: ...}}       query block (complex DSL — do not generate without user input)
{{[[table]]}}            table container block
```

## Tables

Create a `{{[[table]]}}` block, then:

- Each child = a row
- Each grandchild = a cell

Or use pipe tables inside a regular block:

```
| Col A | Col B |
|-------|-------|
| val   | val   |
```

## Headings

Set via `heading` field in batch actions (`0`-`3`), not markdown `#`.

## Queries (reference only)

```
{{[[query]]: {and: [[tag1]] [[tag2]]}}}
{{[[query]]: {between: [[Jan 1st, 2026]] [[today]]}}}
{{[[query]]: {or: [[tag1]] [[tag2]]}}}
{{[[query]]: {not: [[tag]]}}}
```

## Daily Page Date Format

Roam uses `Month Dth, YYYY` with ordinal suffixes.

| Day | Suffix | Example |
|-----|--------|---------|
| 1, 21, 31 | st | `January 1st, 2026`, `March 21st, 2026` |
| 2, 22 | nd | `February 2nd, 2026`, `April 22nd, 2026` |
| 3, 23 | rd | `March 3rd, 2026`, `June 23rd, 2026` |
| 4-20, 24-30 | th | `March 30th, 2026`, `July 15th, 2026` |

**Never use ISO format** (`2026-03-30`). Roam will not find the page.
