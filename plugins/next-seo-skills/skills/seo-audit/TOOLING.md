This file defines how live audit data must be collected.

## Core rule

Prefer **machine-readable** sources only.

Use tools in this order:

1. Direct HTTP fetch
2. HTML parsing
3. `robots.txt` / `sitemap.xml`
4. PageSpeed Insights API

Do not use visual UI as the main source.

---

# 1. Direct HTTP fetch

Use it to collect:

- status code
- redirects
- final URL
- response headers
- raw HTML

This is the primary source of truth for:

- reachability
- indexability
- live metadata output

---

# 2. HTML parsing

Extract at minimum:

- `<title>`
- meta description
- canonical
- robots meta
- Open Graph tags
- Twitter tags if present
- JSON-LD if present

Do not infer SEO output from visible text alone.

---

# 3. robots.txt and sitemap.xml

Fetch:

- `/robots.txt`
- `/sitemap.xml`

Use them only as supporting crawl/index signals.

Do not let them distract from the target URL.

---

# 4. PageSpeed Insights API

Use PSI only if `.env*` contains:

```env
PAGESPEED_INSIGHTS_API=your_api_key_here
```

If the key is missing:

- do not call PSI and alert the user to add PAGESPEED_INSIGHTS_API env and get it from https://developers.google.com/speed/docs/insights/v5/get-started
- continue with fetch + HTML + robots + sitemap only

## Endpoint

```txt
https://pagespeedonline.googleapis.com/pagespeedonline/v5/runPagespeed?url=<ENCODED_URL>&strategy=mobile&category=seo&category=performance&key=<PAGESPEED_INSIGHTS_API>
```

Default:

- `strategy=mobile`

Only request:

- `category=seo`
- `category=performance`

## Extract only these PSI fields

From `lighthouseResult.categories`:

- `seo.score`
- `performance.score`

From `lighthouseResult.audits`:

- `first-contentful-paint`
- `largest-contentful-paint`
- `total-blocking-time`
- `cumulative-layout-shift`
- `speed-index`
- `server-response-time`
- `interactive`

For each metric, keep only:

- `score`
- `displayValue`

From SEO audit refs:

- `id`
- `score`
- `displayValue`
- `description`

Only report:

- failed SEO audits
- non-perfect SEO audits
- key performance metrics

Do not dump full PSI JSON.

Example:

```bash
curl -s "https://pagespeedonline.googleapis.com/pagespeedonline/v5/runPagespeed?url=<ENCODED_URL>&strategy=mobile&category=seo&category=performance&key=$PAGESPEED_INSIGHTS_API" \
| python3 -c "
import sys, json
d = json.load(sys.stdin)

lr = d.get('lighthouseResult', {})
cats = lr.get('categories', {})
audits = lr.get('audits', {})

print('SEO score:', cats.get('seo', {}).get('score'))
print('Perf score:', cats.get('performance', {}).get('score'))

metrics = [
    'first-contentful-paint',
    'largest-contentful-paint',
    'total-blocking-time',
    'cumulative-layout-shift',
    'speed-index',
    'server-response-time',
    'interactive'
]

for m in metrics:
    a = audits.get(m, {})
    print(f'{m}:', a.get('displayValue', 'N/A'), '| score:', a.get('score', 'N/A'))

print('--- SEO AUDITS ---')
seo_refs = cats.get('seo', {}).get('auditRefs', [])

for ref in seo_refs:
    aid = ref.get('id')
    a = audits.get(aid, {})
    sc = a.get('score')

    if sc is None:
        print(f'N/A  [{aid}] score=None')
    elif sc < 1:
        print(f'FAIL [{aid}] score={sc} | {a.get(\"displayValue\", \"\")} | {a.get(\"description\", \"\")}')
"
```

---

# 5. Audit order

Always collect data in this order:

1. Fetch target URL
2. Confirm status, redirects, final URL
3. Parse metadata and canonical
4. Check robots meta / indexability
5. Parse OG / Twitter / JSON-LD
6. Fetch `robots.txt`
7. Fetch `sitemap.xml`
8. Run PSI only if `PAGESPEED_INSIGHTS_API` exists
9. Consolidate findings

---

# 6. Reliability rules

- **Confirmed**: directly visible in HTTP, HTML, robots, sitemap, or PSI
- **Likely**: strongly suggested by live signals
- **Unverified**: cannot be confirmed from live output

Always label findings clearly.

---

# 7. Final rule

The default audit stack is:

- direct HTTP fetch
- HTML parsing
- `robots.txt`
- `sitemap.xml`
- PSI only if `PAGESPEED_INSIGHTS_API` exists
