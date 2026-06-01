---
name: claude-plugin-authoring
description: Basics of authoring a Claude Code plugin — the plugin.json manifest, the .claude-plugin/ directory layout, the component types (skills, agents, commands, hooks, MCP), and the validate/install workflow. Use when creating a new Claude Code plugin or wiring one into a marketplace.
---

# Claude Plugin Authoring (Basics)

A minimal guide to standing up a Claude Code plugin. For the full schema and
edge cases, see the official docs at <https://code.claude.com/docs/en/plugins>.

## 1. Manifest — `.claude-plugin/plugin.json`

Only `name` is required. A minimal manifest:

```json
{ "name": "my-plugin", "version": "0.1.0", "description": "..." }
```

The manifest lives at `.claude-plugin/plugin.json`. Component-path fields
(`skills`, `agents`, `commands`, `hooks`, `mcpServers`) are **overrides** — omit
them when you use the default directory layout below and let Claude Code
auto-discover.

## 2. Directory layout

```
my-plugin/
├── .claude-plugin/
│   ├── plugin.json          ← manifest (only this + marketplace.json go here)
│   └── marketplace.json     ← only if self-publishing as a marketplace
├── skills/<name>/SKILL.md   ← skills, auto-discovered
├── agents/*.md              ← sub-agent definitions
├── commands/*.md            ← legacy flat commands (prefer skills/ for new work)
├── hooks/hooks.json         ← hooks (event-keyed object, not a flat array)
└── README.md
```

**Footgun:** only `plugin.json` (and `marketplace.json`) belong inside
`.claude-plugin/`. Putting `skills/`, `agents/`, `hooks/`, etc. inside it is a
silent loader failure.

## 3. Components at a glance

| Component   | Location                 |
| ----------- | ------------------------ |
| Skills      | `skills/<name>/SKILL.md` |
| Agents      | `agents/*.md`            |
| Commands    | `commands/*.md` (legacy) |
| Hooks       | `hooks/hooks.json`       |
| MCP servers | `.mcp.json`              |

A `SKILL.md` needs only `name` and `description` frontmatter. Front-load the
`description` with concrete trigger phrases — Claude under-triggers skills whose
descriptions are vague.

## 4. Path & variable rules

- All plugin-internal paths must be relative and start with `./` — files
  outside the plugin dir are **not** copied into the install cache.
- Use `${CLAUDE_PLUGIN_ROOT}` for paths to bundled scripts; use
  `${CLAUDE_PLUGIN_DATA}` for persistent state that must survive updates.

## 5. Validate & install

```bash
claude plugin validate          # check manifest + component frontmatter
claude --plugin-dir ./my-plugin # load locally without installing
```

Publish by adding an entry to a marketplace's `marketplace.json`, then:

```bash
claude plugin marketplace add <owner>/<marketplace-repo>
claude plugin install <plugin-name>@<marketplace-name>
```

**Versioning:** if `version` is set in `plugin.json`, you must bump it for users
to receive updates — pushing commits at the same version is a no-op. Omit
`version` while iterating to let the git SHA drive updates.
