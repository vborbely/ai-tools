# WordPress Audit Report: [Site Name]

**Audit Date:** [Date]
**URL:** [URL]
**Audit Mode:** Black-box (unauthenticated) | Authenticated (MCP-enriched)
**WordPress Confirmed:** [Confirmed | Likely] — core version [X.Y] ([source], [confidence])
**Pages Analyzed:** [Count]
**Authorization:** Audit performed with the owner's authorization. All checks were passive / low-impact reads.

---

## Executive Summary

**Overall WP Health Score: [X]/100 ([Rating])**

[2-3 sentences: biggest risks, overall posture, the single most urgent fix.]

### Score Breakdown

| Category | Score | Weight | Weighted |
|---|---|---|---|
| Attack-Surface Exposure | [X]/100 | 30% | [X] |
| Component Currency | [X]/100 | 20% | [X] |
| Hardening & TLS | [X]/100 | 15% | [X] |
| Performance & Delivery | [X]/100 | 15% | [X] |
| SEO & Discoverability | [X]/100 | 10% | [X] |
| Best Practices & Hygiene | [X]/100 | 10% | [X] |
| **Overall** | | | **[X]/100** |

---

## Critical Issues (Fix Immediately)
[For each: title, evidence URL, why it matters, exact fix. e.g. "Exposed wp-config.php.bak at <url> leaks DB credentials — delete the file and rotate DB password + salts."]

## High Priority (Within 1 Week)
[...]

## Medium Priority (Within 1 Month)
[...]

## Low Priority (When Possible)
[...]

---

## Category Deep Dives

### Attack-Surface Exposure ([X]/100)
[Probe-by-probe results table: probe / result / evidence / severity.]

### Component Currency ([X]/100)
[Inventory table: type / slug / detected version / latest / behind / known vuln / confidence.]

### Hardening & TLS ([X]/100)
[Header coverage, TLS details, cookie flags, version leaks.]

### Performance & Delivery ([X]/100)
[Weight, requests, caching, CDN, CWV proxies.]

### SEO & Discoverability ([X]/100)
[robots/sitemap/canonical/schema/indexability.]

### Best Practices & Hygiene ([X]/100)
[Debug off, no leaked paths, misc.]

---

## Quick Wins (This Week)
1. [Specific action + expected impact]
2. ...

## 30-Day Action Plan
### Week 1: [Theme]
- [ ] ...
### Week 2: [Theme]
- [ ] ...
### Week 3: [Theme]
- [ ] ...
### Week 4: [Theme]
- [ ] ...

---

## Limitations of Black-Box Audit
This audit observed only the public surface. It cannot see deactivated plugins, exact installed versions for all components, server/file integrity, or admin configuration. For a definitive inventory and integrity check, run the **authenticated mode** (this plugin's bundled `mcp/` server).

## Appendix: Endpoints Checked
| Endpoint | Status | Note |
|---|---|---|
| [path] | [code] | [observation] |
