# Changelog

All notable changes to the **coos** (Company OS) plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> **Why this matters for delivery:** the version below must always match the
> `version` field in `.claude-plugin/plugin.json`. Because `version` is set,
> installed users only receive an update when that number is **bumped** —
> pushing commits without bumping it has no effect. Bumping the version is the
> release signal.

## How to release

1. Make your changes to a skill (`skills/<skill-name>/…`).
2. Move items from **[Unreleased]** into a new dated version section below.
3. Bump `version` in `plugins/coos/.claude-plugin/plugin.json` to match
   (MAJOR.MINOR.PATCH — breaking / feature / fix).
4. Commit and tag:
   ```bash
   git commit -am "coos X.Y.Z: <summary>"
   git tag coos-vX.Y.Z
   git push --follow-tags
   ```
5. Users pick it up via `/plugin marketplace update ai-tools` →
   `/plugin update coos@ai-tools` → `/reload-plugins` (or automatically if
   they enabled auto-update for the marketplace).

---

## [Unreleased]
### Added
- Initial addition: `workflow-def`, an interactive "grill me"-style workflow/process
  documentation interviewer for a Company OS knowledge base. Interviews a colleague
  about one workflow, captures it as atomic steps, proposes de-facto best-practice
  steps that may be missing (via bundled `playbooks/*.md` references), and writes
  the result into a Gitea-hosted Company OS repo on a branch and Pull Request
  (never directly to `main`). Requires a connected Gitea MCP server.

[Unreleased]: https://github.com/vborbely/ai-tools/compare/coos-v0.1.0...HEAD
