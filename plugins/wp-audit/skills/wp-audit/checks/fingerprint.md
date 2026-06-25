# Check: WordPress Confirmation & Fingerprinting

Goal: confirm the target runs WordPress and identify the core version, with a confidence level for each signal (signals are spoofable).

## 1. Confirm WordPress

Fetch the homepage and a couple of internal pages. Look for any of:

| Signal | How to detect | Strength |
|---|---|---|
| Asset paths | `/wp-content/`, `/wp-includes/` in HTML `src`/`href` | Strong |
| Generator meta | `<meta name="generator" content="WordPress X.Y" />` | Strong (spoofable) |
| REST API link | `<link rel="https://api.w.org/" href=".../wp-json/">` or `Link:` header | Strong |
| Login surface | `GET /wp-login.php` → 200 with WP login form | Strong |
| XML-RPC | `GET /xmlrpc.php` → "XML-RPC server accepts POST requests only" | Strong |
| Emoji/script | `wp-emoji-release.min.js`, `wp-embed.min.js` | Medium |
| Feed generator | `/feed/` → `<generator>https://wordpress.org/?v=X.Y</generator>` | Medium |

Output a confidence: **Confirmed** (≥2 strong), **Likely** (1 strong or several medium), **Not WordPress** (none). If Not WordPress, stop the audit and say so.

## 2. Core Version Detection

Try in this order; record value + source + confidence:

1. `GET /wp-json/` — sometimes includes version; often the most accurate when exposed.
2. Generator meta tag on homepage / feed.
3. `GET /readme.html` — `Version X.Y` (often removed by hardened sites — its absence is itself a small positive signal).
4. `ver=` query string on **bundled core** assets (e.g. `wp-includes/js/wp-embed.min.js?ver=6.5.2`) — core assets share the core version; plugin assets do not.
5. `/feed/` generator `?v=` parameter.

Note: a missing/spoofed version is common on hardened sites. Mark confidence accordingly and never treat a single banner as authoritative.

## 3. Hosting / Stack Hints (informational)

From response headers: `Server`, `X-Powered-By`, `X-Cache`, CDN headers (`cf-ray`, `x-amz-cf-id`, `x-served-by`). These inform performance/hardening checks but are not security findings on their own (though leaking exact versions is a Low/Medium finding — see hardening check).

## Output

```
wordpress_confirmed: Confirmed | Likely | No
core_version: { value, source, confidence }
stack_hints: { server, cdn, powered_by }
```
