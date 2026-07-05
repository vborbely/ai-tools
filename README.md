# AI Tools

A Claude Code **plugin marketplace** of AI-native tools — skills and (soon) MCP servers for web auditing and optimization.

## Install

```bash
# In Claude Code, add this repo as a marketplace, then install a plugin:
/plugin marketplace add vborbely/ai-tools      # replace with your published repo
/plugin install wp-audit@ai-tools
/plugin install coos@ai-tools
```

> Relative plugin sources resolve only when the marketplace is added via a **git** source (GitHub/GitLab/git URL), not a raw URL.

## Plugins

| Plugin | What it does |
|---|---|
| **[wp-audit](plugins/wp-audit/)** | Black-box WordPress security & health auditor. Fingerprints core/plugins/themes, probes the public attack surface (XML-RPC, user enumeration, exposed files), checks hardening/TLS/performance/SEO, and produces a scored report with a prioritized remediation plan. An authenticated MCP deep-scan mode is planned. |
| **[coos](plugins/coos/)** | Company OS toolkit. `workflow-def`: an interactive interviewer that grills a colleague about one workflow, cross-checks it against best-practice playbooks, and writes the result into a Gitea-hosted Company OS knowledge base as a branch + Pull Request. Requires a connected Gitea MCP server; an optional `coos-mcp` MCP server speeds up duplicate-document checks. |

## Usage

**wp-audit** — give Claude Code a WordPress site URL, or say "audit this WordPress site":

```
Audit https://example.com for security and health issues.
```

**coos / workflow-def** — say "document this workflow" or "grill me about our process", then answer the interview questions. The skill needs a Gitea MCP server connected (to write the branch + PR); if a `coos-mcp` MCP server is also connected, it's used automatically to speed up the duplicate-document check:

```
Document how we onboard a new client.
```

The skill interviews you step by step, drafts the workflow into the target Company OS repo on a new branch, and opens a Pull Request for review — it never commits directly to `main`.

## Repository layout

```
ai-tools/                         # marketplace repo
├── .claude-plugin/
│   └── marketplace.json          # lists the plugins below
└── plugins/
    ├── wp-audit/                 # one self-contained plugin
    │   ├── .claude-plugin/
    │   │   └── plugin.json
    │   ├── skills/
    │   │   └── wp-audit/         # SKILL.md + checks/ + report/  (one level deep — required)
    │   └── mcp/                  # planned authenticated MCP server (+ .mcp.json when built)
    └── coos/                     # Company OS toolkit plugin
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── workflow-def/     # SKILL.md + playbooks/ (8 archetypes) + template/
```

New tools are added as additional entries under `plugins/` and one line in `marketplace.json`.

## License

MIT (per-plugin `plugin.json`).
