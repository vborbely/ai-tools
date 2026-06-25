# Check: Hardening, Headers & TLS

Goal: assess transport security, security headers, cookie flags, and version leakage. Score 0-100 (higher = better hardened).

## Security Headers

Inspect response headers on the homepage and a sample inner page:

| Header | Expected | Weight | Missing severity |
|---|---|---|---|
| `Strict-Transport-Security` | `max-age>=15552000; includeSubDomains` | 20 | High (on HTTPS site) |
| `Content-Security-Policy` | present, not trivially `unsafe-inline` everywhere | 20 | Medium |
| `X-Frame-Options` / CSP `frame-ancestors` | `SAMEORIGIN`/`DENY` | 15 | Medium |
| `X-Content-Type-Options` | `nosniff` | 10 | Medium |
| `Referrer-Policy` | `strict-origin-when-cross-origin` or stricter | 10 | Low |
| `Permissions-Policy` | present | 10 | Low |
| `Cross-Origin-Opener-Policy` | present (bonus) | 5 | Low |

## TLS

- HTTPS reachable; HTTP (`http://`) **redirects** to HTTPS (301/308).
- Certificate valid, not expired, hostname matches; note days-to-expiry (<21 days → flag).
- TLS ≥ 1.2 (prefer 1.3); flag TLS 1.0/1.1 or SSLv3.
- Use `curl -sI` / `curl -vI` (or `openssl s_client -connect host:443`) read-only.

## Cookies

For any `Set-Cookie` (esp. `wordpress_*`, `wp-settings`, session): check `Secure`, `HttpOnly`, `SameSite`. Missing flags on auth-adjacent cookies → Medium.

## Version / Banner Leakage

- `Server:` and `X-Powered-By:` exposing exact versions (e.g. `Apache/2.4.41`, `PHP/7.4.3`) → Medium (older versions) / Low.
- `<meta name="generator" content="WordPress X.Y">` present → Low.

## Scoring

Header score = sum of weights present / total weights × 100, then apply TLS and cookie penalties:
- No HTTPS or HTTP not redirecting: cap score at 40.
- Expired/invalid cert: cap at 30.
- Each missing cookie flag on auth cookie: −5.
- Version banner leak: −5 each (max −10).

## Output

```
headers: { name: present|absent }
tls: { https, redirects, version, cert_valid, days_to_expiry }
cookies: [ { name, secure, httponly, samesite } ]
version_leaks: [ ... ]
hardening_score: 0-100
```
