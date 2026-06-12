# Next.js SEO Fix Patterns

Use these patterns when fixing SEO in Next.js projects.

## 1. Routing detection

- If the project uses `app/`, prefer App Router patterns
- If the project uses `pages/`, prefer Pages Router patterns
- If both exist, preserve the current architecture and avoid cross-router refactors unless necessary

## 2. Metadata source of truth

- Prefer metadata from the same source as page content
- If content comes from API, CMS, or database, derive SEO fields from that source when possible
- Prefer dedicated SEO fields when available:
  - `seoTitle`
  - `seoDescription`
  - `ogImage`
  - `canonicalUrl`
- Fall back in this order:
  1. dedicated SEO fields
  2. content title/excerpt/summary
  3. safe generic fallback

## 3. App Router patterns

Prefer:

- `export const metadata` for static pages
- `generateMetadata` for dynamic pages or fetched data

Use `generateMetadata` for:

- blog post pages
- article pages
- product pages
- category pages
- docs pages
- slug routes
- any route where metadata depends on params, search params, or fetched data

Avoid:

- one root metadata object for all dynamic content
- hardcoded metadata on slug pages
- metadata disconnected from page data

## 4. Pages Router patterns

Prefer:

- page-level `Head` for page-specific metadata
- a shared SEO component only when it reduces repeated code cleanly

Avoid:

- putting page-specific metadata in `_document`
- duplicating large `Head` blocks across many pages when a helper already makes sense
- introducing a large SEO abstraction for a small project

## 5. Canonical patterns

Prefer:

- one site URL helper or config source
- route-aware canonical generation
- canonical values aligned with the public URL strategy

Avoid:

- canonical pointing every page to homepage
- canonical values built from unstable query params by default
- hardcoded production domains unless already established in the project

## 6. Open Graph and Twitter patterns

Prefer:

- page-aware title and description
- route-aware `url`
- stable image URLs
- article/product-specific metadata where relevant

Avoid:

- reusing the same OG data for all dynamic pages
- missing image handling on shareable content
- social metadata that conflicts with canonical intent

## 7. Robots patterns

Prefer:

- a valid production-safe `robots.txt` strategy
- allowing crawl for public pages
- disallowing only clearly private or internal paths
- adding sitemap reference when available

Avoid:

- site-wide blocking by mistake
- preview/staging restrictions leaking into production
- conflicting robots behavior across implementations

## 8. Sitemap patterns

Prefer:

- including all important public canonical pages
- generating sitemap entries from API/CMS/database for dynamic content
- aligning sitemap URLs with canonical URLs

Avoid:

- missing high-value public pages
- including private, preview, duplicate, or non-canonical URLs
- using placeholder domains

## 9. JSON-LD patterns

Prefer:

- page-specific schema
- small schema helpers
- schema derived from real page data

Common schema targets:

- `Organization`
- `WebSite`
- `BreadcrumbList`
- `Article`
- `Product`
- `FAQPage`

Avoid:

- injecting schema that does not match the page
- reusing one schema for unrelated content types
- hardcoding misleading structured data

## 10. Rendering strategy for SEO pages

Prefer:

- static generation for stable public pages
- ISR for public pages that change over time
- request-time rendering only when freshness or request context truly requires it

Guidance:

- blog, docs, category, landing, and many product pages should usually prefer static generation or ISR
- if page content is fetched from API and does not need per-request freshness, prefer ISR over always-dynamic rendering
- if metadata depends on API data, keep metadata and page data consistent in caching strategy

If a Vercel skill is available:

- it may be used to evaluate SSR, SSG, ISR, caching, revalidation, and deployment-aware behavior

Avoid:

- making public SEO pages fully dynamic without a clear reason
- changing render strategy unless the SEO or freshness tradeoff justifies it

## 11. Edit safety

Before editing:

- check for existing SEO utilities
- check for existing site URL helpers
- check for current metadata conventions
- check whether the project already uses a schema helper or SEO component

When editing:

- make the smallest correct change
- preserve naming conventions and folder structure
- reuse existing utilities before creating new ones

Avoid:

- refactoring unrelated code
- inventing business rules
- adding large helper layers for a simple fix

## 12. Fallback behavior

When required information is missing:

- prefer a safe fallback
- add a small TODO only when necessary
- do not invent production URLs, business copy, or SEO data without evidence
