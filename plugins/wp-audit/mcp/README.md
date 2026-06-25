# wp-audit MCP Server (planned) — Authenticated WordPress Audit

**Status: planned / not yet implemented.** This folder marks the home of the authenticated companion to the `wp-audit` skill (`../skills/wp-audit`).

## Why an MCP (and not more skill)

The `wp-audit` skill is **black-box**: it only sees what an anonymous visitor sees. Some of the most valuable audit data requires *authenticated server access*, which a skill alone cannot reach:

- Exact installed plugin/theme list **including deactivated ones** and precise versions
- WordPress user accounts and roles
- File integrity (`wp core verify-checksums`, modified core files)
- Admin/config flags (debug mode, file editing, auto-update status, security plugin presence)
- Database-level checks (table prefix, admin user `id=1`, orphaned data)

MCP is the right tool because it *connects Claude to a system it cannot otherwise reach* (an authenticated WP install / WP-CLI / REST with credentials). The skill remains the orchestrator and report generator.

## Integration Contract

The skill auto-detects whether `wp-audit` MCP tools are available. When present, it calls them in Phase 1/2 and **prefers verified data over black-box inference**, marking those findings `confidence: Verified` and setting report `Audit Mode: Authenticated`.

Planned tool surface (subject to refinement):

| Tool | Returns |
|---|---|
| `wp_core_info` | core version, locale, multisite, auto-update status, file-edit flag, debug flag |
| `wp_component_inventory` | all plugins/themes: slug, version, active/inactive, update-available, auto-update |
| `wp_integrity_check` | `verify-checksums` result, list of modified/unknown core files |
| `wp_user_audit` | users: id, login, role, last-login (no password material) |
| `wp_config_flags` | hardened-config signals: `DISALLOW_FILE_EDIT`, `FORCE_SSL_ADMIN`, table prefix, security plugins active |

Each tool returns JSON shaped to drop into the corresponding skill check's input (see `../skills/wp-audit/checks/*.md` "Note for Authenticated Mode").

## Likely Implementation

- A thin MCP server (Node or Python) that connects to the target site via **WP-CLI over SSH** and/or an **authenticated WP REST** application password.
- Credentials provided via MCP server config / env — never hard-coded, never logged.
- Read-only operations only; no mutation of the target site.

## Distribution Note

Because an MCP server is heavier to install than a skill (users must run a process and supply credentials), this stays an **optional power-user add-on**. The skill is the primary, low-friction community deliverable; this MCP is for those who need authenticated deep-scans.
