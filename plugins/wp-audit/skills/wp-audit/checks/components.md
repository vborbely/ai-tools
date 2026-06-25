# Check: Component Inventory & Currency (Plugins / Themes)

Goal: enumerate installed plugins and themes (black-box) and flag outdated or known-vulnerable components. Score 0-100 (higher = more current/fewer known vulns).

## Enumeration (black-box)

Detected from public surface only — this finds *active/asset-loading* components, not deactivated ones (a key black-box limitation; the MCP authenticated mode returns the full list).

1. **Asset paths** in page HTML:
   - Plugins: `/wp-content/plugins/<slug>/...`
   - Themes: `/wp-content/themes/<slug>/...`
   Collect every distinct `<slug>`.
2. **Versions** per slug, in order of reliability:
   - `/wp-content/plugins/<slug>/readme.txt` → `Stable tag:` (most reliable when present)
   - `ver=` query string on that component's enqueued assets (often the real version; sometimes the core version — note ambiguity)
   - `/wp-content/plugins/<slug>/<slug>.php` header comment (if readable)
3. **Themes**: `/wp-content/themes/<slug>/style.css` → `Version:` header.

## Known-Vulnerability Mapping

For each `(slug, version)`:
- Compare against the latest stable release to compute "versions behind," using the WordPress.org directory APIs:
  - **Plugins:** `https://api.wordpress.org/plugins/info/1.0/<slug>.json` → `.version`
  - **Themes:** `https://api.wordpress.org/themes/info/1.2/?action=theme_information&request%5Bslug%5D=<slug>` → `.version`
    URL-encode the `request[slug]` bracket as `request%5Bslug%5D` — an unencoded `[` makes `curl` fail with "bad range in URL". Quote the URL in shells regardless.
  - A `404`/empty response means the component is **not in the public directory** (premium/custom) — report `latest: unknown (premium/custom)`, not "outdated."
- Flag components with **known CVEs/WPScan advisories** at or below the detected version. Severity follows the advisory: RCE / auth-bypass / SQLi → Critical; XSS/CSRF/info-disclosure → High; lower-impact → Medium.
- When no advisory data is reachable, fall back to "outdated by N major/minor versions" as a Medium signal.

> Do not fabricate CVE IDs. If you cannot verify an advisory, report it as "outdated; manual vuln-check recommended" rather than asserting a specific CVE.

## Scoring

- Start at 100.
- −25 per component with a Critical-rated known vuln.
- −12 per component with a High-rated known vuln.
- −5 per component outdated by a major version with no known vuln.
- −2 per component outdated by a minor version.
- Floor at 0.

## Output

```
components: [ { type: plugin|theme, slug, detected_version, latest_version,
                versions_behind, known_vuln: {severity, ref} | null,
                evidence_url, confidence } ]
currency_score: 0-100
```

## Note for Authenticated Mode

When the bundled MCP server (`../../mcp/`) is present, replace inferred versions with the exact installed list (including deactivated plugins) and set `confidence: Verified`.
