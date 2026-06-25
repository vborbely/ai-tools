# Check: Performance & Delivery

Goal: assess delivery efficiency and Core Web Vitals proxies (black-box, no field data). Score 0-100 (higher = faster/leaner).

## Signals

| Signal | How | Good |
|---|---|---|
| Page weight | Sum of HTML + referenced CSS/JS/image bytes (head requests / content-length) | < 2 MB total |
| Requests | Count of distinct asset URLs in HTML | < 60 |
| Render-blocking | CSS/JS in `<head>` without `defer`/`async`/`media` | minimal |
| Caching | `Cache-Control: max-age`, `ETag`, `Expires` on static assets | long max-age on static |
| CDN | CDN headers (`cf-ray`, `x-amz-cf-id`, `x-served-by`, `x-cache`) | present |
| Compression | `Content-Encoding: gzip|br` | brotli/gzip on text |
| Image optimization | modern formats (`.webp`/`.avif`), dimensions set, lazy-loading | present |
| HTTP version | `HTTP/2` or `HTTP/3` | yes |

## Core Web Vitals proxies

No real field data without RUM; estimate qualitatively:
- **LCP proxy:** large hero image weight + render-blocking CSS before it.
- **CLS proxy:** images/iframes without explicit dimensions.
- **INP proxy:** total JS payload + number of third-party scripts (INP replaces FID).

Optionally call the PageSpeed Insights API if the user provides a key; otherwise report proxy-based estimates and say so.

## Scoring

Composite of: page weight (25), request count (15), caching+compression (20), CDN+HTTP/2-3 (15), render-blocking (15), image optimization (10). Normalize to 0-100.

## Output

```
performance: { total_bytes, request_count, render_blocking_count,
               caching, cdn, compression, http_version, cwv_proxies }
performance_score: 0-100
```
