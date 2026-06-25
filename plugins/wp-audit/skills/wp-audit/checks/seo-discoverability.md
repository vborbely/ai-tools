# Check: SEO & Discoverability

Goal: verify the fundamentals that let search engines and AI crawlers find and understand the site. Score 0-100 (higher = better). Complements (does not duplicate) the `geo-*` skills — keep this to WordPress-relevant basics.

## Signals

| Signal | How | Good |
|---|---|---|
| `robots.txt` | `GET /robots.txt` | present, not blocking the whole site, sitemap referenced |
| Sitemap | `/sitemap.xml`, `/sitemap_index.xml`, or WP core `/wp-sitemap.xml` | present, valid, reachable |
| Indexability | meta robots / `X-Robots-Tag` | not `noindex` on public pages |
| Canonical | `<link rel="canonical">` on pages | present, self-referential, consistent |
| Titles & meta | `<title>`, `<meta name="description">` | unique, present per page |
| Open Graph / Twitter | `og:*`, `twitter:card` | present |
| Headings | one `<h1>`, logical hierarchy | yes |
| Schema.org | JSON-LD blocks (Organization, Article, etc.) | present, valid |
| Hreflang | if multilingual | correct |
| Permalinks | pretty permalinks vs `?p=123` | pretty |
| Mixed content | http assets on https page | none |

## Notes

- WordPress core ships `/wp-sitemap.xml` since 5.5 unless overridden by an SEO plugin (Yoast/RankMath use their own). Detect which.
- Accidental `noindex` (a common "launched but still in maintenance mode" mistake) is a High finding here.

## Scoring

Weighted coverage of the signals above, normalized to 0-100. A site-wide `noindex` caps the score at 30.

## Output

```
seo: { robots_txt, sitemap, indexable, canonical, og, schema, permalinks, mixed_content }
seo_score: 0-100
```
