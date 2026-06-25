# AI Tools

A Claude Code **plugin marketplace** of AI-native tools — skills and (soon) MCP servers for web auditing and optimization.

## Install

```bash
# In Claude Code, add this repo as a marketplace, then install a plugin:
/plugin marketplace add vborbely/ai-tools      # replace with your published repo
/plugin install wp-audit@ai-tools
```

> Relative plugin sources resolve only when the marketplace is added via a **git** source (GitHub/GitLab/git URL), not a raw URL.

## Plugins

| Plugin | What it does |
|---|---|
| **[wp-audit](plugins/wp-audit/)** | Black-box WordPress security & health auditor. Fingerprints core/plugins/themes, probes the public attack surface (XML-RPC, user enumeration, exposed files), checks hardening/TLS/performance/SEO, and produces a scored report with a prioritized remediation plan. An authenticated MCP deep-scan mode is planned. |

## Repository layout

```
ai-tools/                         # marketplace repo
├── .claude-plugin/
│   └── marketplace.json          # lists the plugins below
└── plugins/
    └── wp-audit/                 # one self-contained plugin
        ├── .claude-plugin/
        │   └── plugin.json
        ├── skills/
        │   └── wp-audit/         # SKILL.md + checks/ + report/  (one level deep — required)
        └── mcp/                  # planned authenticated MCP server (+ .mcp.json when built)
```

New tools are added as additional entries under `plugins/` and one line in `marketplace.json`.

## License

MIT (per-plugin `plugin.json`).
