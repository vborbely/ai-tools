# Check: Attack-Surface Exposure

Goal: identify publicly exposed endpoints, files, and enumeration vectors. All checks are **read-only, single-request, non-destructive**. Score 0-100 (higher = less exposed).

> Authorization required. Only run against sites the user owns or is authorized to assess.

## Probes

| # | Probe | Request | Exposure if... | Severity |
|---|---|---|---|---|
| 1 | XML-RPC enabled | `POST /xmlrpc.php` body `<methodCall><methodName>system.listMethods</methodName></methodCall>` | response lists methods incl. `pingback.ping` / `system.multicall` | High |
| 2 | REST user enumeration | `GET /wp-json/wp/v2/users` | returns JSON array of users (id, slug, name) | High |
| 3 | Author-archive enumeration | `GET /?author=1` (and 2,3) | redirects to `/author/<username>/` revealing login slug | High |
| 4 | wp-config backups | `GET` of `wp-config.php.bak`, `wp-config.php~`, `wp-config.php.save`, `wp-config.php.txt`, `.wp-config.php.swp` | any returns 200 with PHP/credentials | Critical |
| 5 | Debug log exposed | `GET /wp-content/debug.log` | 200 with log content | Critical |
| 6 | Uploads dir listing | `GET /wp-content/uploads/` | 200 with directory index | Medium–High |
| 7 | Exposed `.git` | `GET /.git/HEAD` | 200 / `ref: refs/heads/...` | Critical |
| 8 | DB dumps / backups | `GET` common names: `/backup.sql`, `/database.sql`, `/*.sql`, `/wp-content/backup-db/`, `/backwpup*` | 200 | Critical |
| 9 | Readme version leak | `GET /readme.html` | 200 with core version | Low |
| 10 | Login page reachable & unthrottled | `GET /wp-login.php` | 200 (note: presence is normal; flag only absence of any protection signals) | Info/Low |
| 11 | Full-path disclosure | trigger a benign error (e.g. `GET /wp-content/plugins/<known>/<file>.php` direct) | absolute server path in output | Medium |
| 12 | User-content PHP exec | check `/wp-content/uploads/` for `.php` (do not upload) | listed/executable PHP in uploads | High |

For probes that could be intrusive (4, 8), limit to a short, well-known candidate list; do not brute-force.

## Scoring

- Start at 100.
- Each Critical exposure: −30.
- Each High: −15.
- Each Medium: −8.
- Each Low/Info: −3.
- Floor at 0.

## Output

```
exposures: [ { probe, result: exposed|clean|unknown, evidence_url,
               severity, detail } ]
exposure_score: 0-100
```
