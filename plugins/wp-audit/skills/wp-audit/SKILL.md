---
name: wp-audit
description: Black-box WordPress security and health audit from a public URL. Fingerprints WordPress core, plugins, and themes; probes the exposed attack surface (XML-RPC, REST user enumeration, exposed sensitive files); checks hardening headers, TLS, performance, and SEO discoverability. Produces a composite WP Health Score (0-100) with a severity-ranked, prioritized remediation plan. Use when the user says "wp audit", "wordpress audit", "audit this wordpress site", "scan a wp site", or provides a URL described as a WordPress site.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - WebFetch
  - Write
---

# WordPress Black-Box Audit Skill

## Purpose

This skill performs a **black-box** (unauthenticated, public-facing) audit of a WordPress site from its URL alone — no admin credentials, no server access, no installed agent. It observes only what any anonymous visitor or attacker can observe over HTTP(S), then scores the site's security posture, component currency, hardening, performance, and discoverability, and produces an actionable remediation plan.

## Scope and Boundaries

**In scope (this skill):** Everything reachable over public HTTP(S) — page HTML, response headers, `robots.txt`, `/wp-json/` REST API, `/xmlrpc.php`, well-known sensitive paths, TLS configuration, and rendered content.

**Out of scope (future MCP — see the bundled `mcp/` server):** Anything requiring authentication or server access — `wp-admin` settings, the real installed-plugin list with exact versions, user roles, database, file integrity (`wp core verify-checksums`), and WP-CLI output. This skill is deliberately designed so that an **authenticated MCP server** can later supply that privileged data and the *same scoring + report pipeline* consumes it. See [Authenticated Mode Hand-off](#authenticated-mode-hand-off) below.

**Ethics and authorization:** This is a defensive/diagnostic audit. Only probe sites the user owns or is authorized to assess. All checks are **passive or low-impact reads** — no exploitation, no brute force, no credential testing, no destructive requests. Respect `robots.txt`, rate-limit requests, and never attempt to bypass access controls.

---

## Audit Workflow

### Phase 1: Confirm WordPress and Fingerprint

See `checks/fingerprint.md` for full detection logic.

1. **Confirm it's WordPress.** Fetch the homepage and look for: `/wp-content/` and `/wp-includes/` asset paths, `<meta name="generator" content="WordPress X.Y">`, `wp-json` link header/`<link rel="https://api.w.org/">`, `wp-emoji`, `/wp-login.php` reachability, `xmlrpc.php` presence. Record confidence (Confirmed / Likely / Not WordPress). If Not WordPress, stop and report.
2. **Detect core version** from (in order of reliability): `/wp-json/` (often hidden), generator meta, `readme.html`, `/feed/` generator tag, and `ver=` query strings on bundled core assets (`wp-includes/js/*`). Record the version and how it was obtained (some are spoofable).
3. **Capture the crawl set.** Fetch homepage + up to 25 internal pages (prioritize: posts, pages, `/wp-login.php`, `/wp-json/`, sitemap entries). Respect `robots.txt`. Enforce per-fetch timeout and inter-request delay (see Quality Gates).

### Phase 2: Parallel Subagent Delegation

Delegate to specialized subagents, each consuming the Phase 1 data and returning a **category score (0-100) + findings list** (each finding: title, severity, evidence/URL, recommended fix). Run these in parallel.

**Subagent 1: Component Inventory & Currency** — `checks/components.md`
Enumerate plugins and themes from asset URLs (`/wp-content/plugins/<slug>/`, `/wp-content/themes/<slug>/`), readme/changelog files, and `ver=` strings. Map detected versions against known-vulnerability data (WPScan/CVE patterns described in the check file). Score on outdated-component count and presence of components with known CVEs.

**Subagent 2: Attack-Surface Exposure** — `checks/security-exposure.md`
Probe (read-only) for: XML-RPC enabled (`xmlrpc.php` → `system.listMethods`), REST user enumeration (`/wp-json/wp/v2/users`), author-archive enumeration (`/?author=1`), exposed `wp-config.php` backups (`.bak`, `~`, `.save`, `wp-config.php.txt`), directory listing on `/wp-content/uploads/`, debug log exposure (`/wp-content/debug.log`), readable `/.git/`, exposed `/wp-content/backup*`/`/*.sql`. Score on count and severity of exposures.

**Subagent 3: Hardening, Headers & TLS** — `checks/hardening-headers.md`
Check security headers (HSTS, CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy), cookie flags, TLS version/cert validity/expiry, HTTP→HTTPS redirect, `Server`/`X-Powered-By` version leakage, login surface hardening signals (login throttling hints, custom login URL). Score on header coverage and TLS posture.

**Subagent 4: Performance & Delivery** — `checks/performance.md`
Assess page weight, render-blocking assets, caching headers (`Cache-Control`, `ETag`), CDN presence, image optimization, and Core Web Vitals proxies (INP-aware). Score on delivery efficiency.

**Subagent 5: SEO & Discoverability** — `checks/seo-discoverability.md`
Check sitemap, `robots.txt`, canonical tags, title/meta/OG/Twitter cards, heading hierarchy, schema.org markup, and indexability. Score on discoverability fundamentals.

> **Single-agent fallback:** If subagent delegation is unavailable, work through each `checks/*.md` file sequentially against the Phase 1 data.

### Phase 3: Score Aggregation & Report

Compute the composite score (below), classify every finding by severity, and write the report using `report/REPORT-TEMPLATE.md`.

---

## Composite WP Health Score

Weighted average of six category scores (0-100):

| Category | Weight | What It Measures |
|---|---|---|
| **Attack-Surface Exposure** | 30% | Publicly exposed endpoints, files, and enumeration vectors |
| **Component Currency** | 20% | Outdated/vulnerable core, plugins, themes |
| **Hardening & TLS** | 15% | Security headers, cookie flags, TLS, version leakage |
| **Performance & Delivery** | 15% | Page weight, caching, CDN, Core Web Vitals proxies |
| **SEO & Discoverability** | 10% | Sitemap, canonical, schema, indexability |
| **Best Practices & Hygiene** | 10% | Misc. config hygiene (debug off, no leaked paths, etc.) |

**Formula:**
```
WP_Health = (Exposure * 0.30) + (Currency * 0.20) + (Hardening * 0.15)
          + (Performance * 0.15) + (SEO * 0.10) + (Hygiene * 0.10)
```

> Note: each category score is "higher is better." For Exposure, an unprobed/clean surface scores high; each confirmed exposure subtracts.

### Score Interpretation

| Range | Rating | Interpretation |
|---|---|---|
| 90-100 | Excellent | Well-hardened; minimal public attack surface |
| 75-89 | Good | Solid posture with a few gaps to close |
| 60-74 | Fair | Notable weaknesses; prioritize the High findings |
| 40-59 | Poor | Multiple serious exposures; remediate promptly |
| 0-39 | Critical | Severe, actively exploitable exposure likely |

---

## Issue Severity Classification

### Critical (Fix Immediately)
- Exposed `wp-config.php` (or readable backup) leaking DB credentials/salts
- Readable `/.git/`, `.sql` dump, or backup archive in webroot
- Core/plugin/theme with a known **actively exploited / RCE / auth-bypass** CVE at the detected version
- Debug log (`debug.log`) exposing paths, queries, or secrets
- Directory listing exposing uploads/backups

### High (Fix Within 1 Week)
- XML-RPC enabled with `system.multicall`/`pingback` (amplification & brute-force vector)
- REST or author-archive **user enumeration** exposed
- Outdated core or plugins with known (non-RCE) vulnerabilities
- No HTTPS, expired cert, or HTTP not redirecting to HTTPS
- Missing HSTS on an HTTPS site

### Medium (Fix Within 1 Month)
- Several missing security headers (CSP, X-Frame-Options, etc.)
- Core/plugin/theme outdated but no known CVE
- Server/version banners leaking software versions
- Cookies missing `Secure`/`HttpOnly`/`SameSite`
- No caching/CDN; heavy render-blocking assets

### Low (Optimize When Possible)
- Generator meta version disclosure
- Missing `Permissions-Policy`/`Referrer-Policy`
- Minor SEO/schema gaps
- Non-critical performance optimizations

---

## Output Format

Write `WP-AUDIT-REPORT.md` using `report/REPORT-TEMPLATE.md`. The report must include: executive summary with composite score + breakdown table, findings grouped by severity (with evidence URLs and concrete fixes), per-category deep dives, a "Quick Wins this week" list, and a 30-day action plan.

---

## Quality Gates

- **Authorization:** Confirm the user owns or is authorized to audit the target before probing. State this assumption in the report.
- **Passive/low-impact only:** No exploitation, brute force, credential testing, or destructive requests. Every probe is a benign GET/HEAD or a single read-only POST (e.g. `system.listMethods`).
- **Page limit:** Max 25 pages per audit; prioritize high-signal endpoints.
- **Timeout:** 30s max per fetch; skip and log on timeout.
- **Rate limiting:** ≥1s between requests; never parallelize probes against a single host aggressively.
- **Robots.txt:** Respect it for crawling. (Well-known security endpoints like `/xmlrpc.php` are probed as single targeted checks, not crawled, and only with authorization.)
- **Spoofing awareness:** Treat all version/banner data as potentially spoofed; mark confidence levels.
- **Graceful failure:** Log failed fetches and continue; report what could not be determined rather than guessing.

---

## Authenticated Mode Hand-off

This skill is the **black-box half** of a two-part design. A future authenticated MCP server (bundled under `mcp/` in this plugin) will expose privileged data that this skill cannot obtain from outside. To keep the skill and MCP decoupled but composable:

- **Contract:** The MCP server should return data shaped like the Phase 1/Phase 2 inputs — a component inventory (exact slugs + versions), core version, user list, file-integrity results, and active config flags. When that data is present, the subagents prefer it over inferred/black-box values and raise their confidence to "Verified."
- **Detection:** At Phase 1, check whether `wp-audit` MCP tools are available (via tool discovery). If yes, call them to enrich/verify; if no, run pure black-box (this skill's default).
- **Report:** The same `report/REPORT-TEMPLATE.md` is used in both modes; an "Audit Mode: Black-box | Authenticated" field in the header distinguishes them, and Verified findings are marked as such.
- **No coupling:** This skill must run fully standalone with zero MCP present. The MCP is purely additive. See `../../mcp/README.md` for the planned tool surface.
