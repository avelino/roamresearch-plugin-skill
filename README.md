# Roam Research - Agent Skill

Agent skill for working with Roam Research graphs via MCP server. Teaches the agent the idiossincrasias do Roam que ele vai errar sem contexto — batch actions, date format, `((uid))` vs `[[page]]`, and Roam-flavored Markdown.

## What the agent learns

- `roam_process_batch_actions` is the canonical write tool (v1.36.0+); individual tools are deprecated
- Daily page titles use `March 30th, 2026` format, not ISO dates
- `((uid))` = block reference (9-char), `[[title]]` = page link — never swap
- Always load the Roam Markdown Cheatsheet via `roam_get_resource` before writing
- Always fetch before creating to avoid duplicates

## Prerequisites

- A coding agent supporting the [Agent Skills standard](https://agentskills.io) (Claude Code, OpenAI Codex, or Windsurf)
- The [roam-tui MCP server](https://github.com/avelino/roam-tui) configured and running (see below)

## MCP Server Setup

This skill requires the [roam-tui](https://github.com/avelino/roam-tui) MCP server — it exposes your Roam graph to AI assistants via the [Model Context Protocol](https://modelcontextprotocol.io/).

### 1. Get a Roam API token

Go to **Roam → Settings → Graph → API tokens** and create a new token.

### 2. Configure the MCP server

Add the following to your MCP client config:

**Claude Code** (`.mcp.json` in your project root or `~/.claude.json` globally):

```json
{
  "mcpServers": {
    "roam": {
      "command": "npx",
      "args": ["-y", "roam-tui@latest", "--mcp"],
      "env": {
        "ROAM_GRAPH_NAME": "your-graph-name",
        "ROAM_GRAPH_API__TOKEN": "roam-graph-token-XXXXX"
      }
    }
  }
}
```

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "roam": {
      "command": "npx",
      "args": ["-y", "roam-tui@latest", "--mcp"],
      "env": {
        "ROAM_GRAPH_NAME": "your-graph-name",
        "ROAM_GRAPH_API__TOKEN": "roam-graph-token-XXXXX"
      }
    }
  }
}
```

If you have `roam-tui` installed locally (via `cargo install` or from source), you can use `"command": "roam", "args": ["--mcp"]` instead.

> Full documentation: [github.com/avelino/roam-tui](https://github.com/avelino/roam-tui)

### 3. Verify

After configuring, restart your agent and confirm the `roam_*` tools are available. A quick test:

```
/roam
> search for "test" in my graph
```

## Skill Installation

### Quick Install

```bash
npx skills add https://github.com/avelino/roamresearch-plugin-skill --skill roam
```

### Setup

#### Option 1: Installer Script (Recommended)

```bash
git clone https://github.com/avelino/roamresearch-plugin-skill.git
cd roamresearch-plugin-skill
./install-skill.sh
```

Select your provider and installation target.

#### Option 2: Manual Install

<details>
<summary>Claude Code</summary>

```bash
git clone https://github.com/avelino/roamresearch-plugin-skill.git
cd roamresearch-plugin-skill

mkdir -p your-project/.claude/skills/roam
cp -r .agents/skills/roam/* your-project/.claude/skills/roam/

mkdir -p your-project/.claude/commands
cp .claude/commands/roam.md your-project/.claude/commands/
```
</details>

<details>
<summary>Codex (OpenAI) / Windsurf</summary>

```bash
git clone https://github.com/avelino/roamresearch-plugin-skill.git
cd roamresearch-plugin-skill

mkdir -p your-project/.agents/skills/roam
cp -r .agents/skills/roam/* your-project/.agents/skills/roam/
```
</details>

## Structure

```
roam/
├── SKILL.md                 ← 15 critical rules + tool table + patterns
└── reference/
    ├── batch-actions.md     ← roam_process_batch_actions schema
    └── roam-markdown.md     ← Roam syntax + daily page date format
```

## Usage

After installation, invoke the skill:

| Provider | Command |
|----------|---------|
| Claude Code | `/roam` |
| Codex | `$roam` |
| Windsurf | `@roam` |

## License

MIT
