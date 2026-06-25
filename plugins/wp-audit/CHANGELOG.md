# Changelog

All notable changes to the **wp-audit** plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> **Why this matters for delivery:** the version below must always match the
> `version` field in `.claude-plugin/plugin.json`. Because `version` is set,
> installed users only receive an update when that number is **bumped** —
> pushing commits without bumping it has no effect. Bumping the version is the
> release signal.

## How to release

1. Make your changes to the skill (`skills/wp-audit/…`).
2. Move items from **[Unreleased]** into a new dated version section below.
3. Bump `version` in `plugins/wp-audit/.claude-plugin/plugin.json` to match
   (MAJOR.MINOR.PATCH — breaking / feature / fix).
4. Commit and tag:
   ```bash
   git commit -am "wp-audit X.Y.Z: <summary>"
   git tag wp-audit-vX.Y.Z
   git push --follow-tags
   ```
5. Users pick it up via `/plugin marketplace update ai-tools` →
   `/plugin update wp-audit@ai-tools` → `/reload-plugins` (or automatically if
   they enabled auto-update for the marketplace).

---

## [Unreleased]

## [0.1.0] - 2026-06-25
### Added
- Initial release: black-box (unauthenticated) WordPress auditor skill.
- WordPress confirmation + core version fingerprinting (`checks/fingerprint.md`).
- Plugin/theme inventory and currency check against the WordPress.org APIs (`checks/components.md`).
- Attack-surface probes — XML-RPC, REST/author user enumeration, exposed `wp-config` backups / `.git` / `debug.log` / SQL dumps, directory listing (`checks/security-exposure.md`).
- Hardening, security headers & TLS checks (`checks/hardening-headers.md`).
- Performance & delivery and SEO/discoverability checks (`checks/performance.md`, `checks/seo-discoverability.md`).
- Composite **WP Health Score** (0-100) with severity-ranked findings and a client-ready report template (`report/REPORT-TEMPLATE.md`).
- Documented hand-off contract for a future authenticated MCP deep-scan mode (`mcp/README.md`).

[Unreleased]: https://github.com/vborbely/ai-tools/compare/wp-audit-v0.1.0...HEAD
[0.1.0]: https://github.com/vborbely/ai-tools/releases/tag/wp-audit-v0.1.0
