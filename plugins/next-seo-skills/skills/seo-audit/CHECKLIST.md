Use this checklist to audit **one live URL**.

This file is for **runtime SEO auditing**, not source-code review.

## Core rules

- Audit the exact input URL
- Prefer direct HTTP/API evidence over assumptions
- Prefer machine-readable data over visual UI
- If something cannot be verified, say:
  - `Unable to verify from live output`

---

# 1. URL and reachability

Check:

- status code
- redirects
- final URL
- whether the page is properly reachable

Severity:

- Critical if the page is not properly reachable
- High if redirects or final URL behavior are misleading

---

# 2. SEO signals

Check all live SEO output for the target URL:

- indexability
  - robots meta
  - x-robots-tag if visible
  - whether the page appears indexable

- metadata
  - title
  - meta description
  - canonical

- social metadata
  - Open Graph basics
  - Twitter basics if present

- structured data
  - whether schema exists
  - whether it appears appropriate

- supporting crawl signals
  - robots.txt
  - sitemap.xml

Severity:

- Critical if the page is unintentionally non-indexable
- Critical if canonical points to the wrong page or wrong domain
- High if important metadata is missing, weak, or misleading
- High if robots or sitemap signals clearly break intended indexing
- Medium if social metadata or structured data is weak or missing

---

# 3. Performance and Lighthouse

Check:

- Lighthouse SEO findings
- LCP
- FCP
- CLS
- TTFB
- Speed Index
- major runtime risks affecting SEO or UX

Severity:

- High if Lighthouse shows major SEO issues
- High if performance is poor enough to materially affect UX/SEO
- Medium for weaker or borderline issues

---

# 4. Final judgment

Summarize:

- overall live SEO health of the URL
- strongest signals
- highest-risk issues
- shortest high-impact fixes
- best follow-up action

---

# Severity model

- Critical:
  - not reachable
  - unintentionally non-indexable
  - wrong canonical target
  - robots blocking intended indexing

- High:
  - missing or weak metadata
  - broken social metadata
  - major Lighthouse SEO issues
  - serious runtime performance issues

- Medium:
  - generic metadata
  - weak social setup
  - missing or weak schema
  - partial crawl signal issues

- Low:
  - minor polish improvements
