# wp-audit — WordPress Black-Box Auditor (Claude Skill)

A Claude Code skill that audits a WordPress site from its public URL — no credentials, no server access. It fingerprints WordPress, inventories plugins/themes, probes the public attack surface, and checks hardening, TLS, performance, and SEO, then produces a scored report with a prioritized remediation plan.

## What it does

- ✅ Confirms WordPress & detects core version
- ✅ Enumerates active plugins/themes and flags outdated/known-vulnerable ones
- ✅ Probes (read-only) for XML-RPC, user enumeration, exposed config/backups/`.git`, debug logs, dir listing
- ✅ Checks security headers, TLS, cookie flags, version leakage
- ✅ Assesses performance & Core Web Vitals proxies and SEO fundamentals
- ✅ Produces `WP-AUDIT-REPORT.md` with a composite **WP Health Score (0-100)**

## Usage

In Claude Code:

```
wp audit https://example.com
```

or any of: "audit this wordpress site", "scan a wp site".

## Structure

```
wp-audit/
├── SKILL.md                  # orchestration, scoring, severity, quality gates
├── checks/
│   ├── fingerprint.md        # WP confirmation + version detection
│   ├── components.md         # plugin/theme inventory + vuln mapping
│   ├── security-exposure.md  # attack-surface probes
│   ├── hardening-headers.md  # headers, TLS, cookies
│   ├── performance.md        # delivery + CWV proxies
│   └── seo-discoverability.md
├── report/
│   └── REPORT-TEMPLATE.md
└── README.md
```

## Ethics & Authorization

Defensive/diagnostic use only. **Audit only sites you own or are explicitly authorized to assess.** All checks are passive or low-impact reads — no exploitation, brute force, credential testing, or destructive requests. The skill respects `robots.txt` and rate-limits requests.

## Roadmap: Authenticated Mode (MCP)

This skill is the **black-box half** of a two-part design. An authenticated MCP server (`../../mcp`) will later provide privileged data — exact installed components (including deactivated), user roles, file integrity (`wp core verify-checksums`), and admin config — which the same scoring/report pipeline consumes to produce a *Verified* audit. The skill runs fully standalone without it; the MCP is purely additive. See `SKILL.md` → "Authenticated Mode Hand-off".

## License

[Add a license before publishing — e.g. MIT.]
