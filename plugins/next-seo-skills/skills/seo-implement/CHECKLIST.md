# CHECKLIST.md

Use this checklist to implement SEO in Next.js projects.

This file is for **implementation**, not audit-only review.

Its purpose is to:

- decide what SEO foundations must be implemented
- determine implementation order
- keep changes focused on the highest-value public SEO surfaces
- prevent overbuilding or drifting into unrelated refactors

Always use this checklist together with:

- `skills/shared/SNIPPETS.md`
- `skills/shared/RENDERING_AND_CACHING.md`

---

# Core rules

- Prefer official Next.js patterns over custom SEO abstractions unless the codebase clearly benefits from abstraction
- Prefer codebase evidence over assumptions
- Prefer the smallest correct implementation that covers the important public SEO routes
- Treat App Router and Pages Router differently
- Treat metadata for dynamic content as a first-class implementation concern
- Implement SEO by route ownership, not by random scattered patches
- Build foundations first, then expand route coverage
- Do not turn implementation into a broad cleanup/refactor unless necessary for a correct SEO foundation

---

# 1. Detect scope before implementing

## 1.1 Project type

Identify:

- is this a Next.js project
- is it App Router, Pages Router, or hybrid
- what are the public SEO-critical routes
- what routes should not be indexed

Completion criteria:

- routing model is known
- public SEO surface is mapped
- obvious non-indexable routes are identified

## 1.2 Version and compatibility

Check:

- `package.json`
- installed `next` version
- whether snippets should follow versioned docs or latest docs

Completion criteria:

- Next.js major version is known or explicitly marked as unverified
- snippet choice is version-aware

## 1.3 Existing SEO baseline

Check whether the project already has:

- root metadata
- route metadata
- dynamic metadata
- robots
- sitemap
- OG/Twitter setup
- schema
- domain/base URL utilities

Completion criteria:

- existing SEO baseline is classified as:
  - absent
  - basic
  - partial
  - already strong in some areas

Implementation rule:

- do not replace a stronger existing pattern with a weaker generic one

---

# 2. Metadata foundation

## 2.1 Root metadata

Implement if missing:

- root-level metadata defaults
- title template if appropriate
- default description if appropriate
- default OG/Twitter base values if appropriate
- `metadataBase` if the app needs canonical/alternate/OG URL composition

Completion criteria:

- the project has one clear global metadata foundation
- root ownership is clear
- domain-safe metadata composition is possible

High priority when:

- there is no meaningful metadata layer
- routes currently rely on scattered fallback values

## 2.2 Section metadata

Implement if useful:

- nested layout metadata for clear sections such as:
  - blog
  - docs
  - marketing
  - product/catalog
  - location/service sections

Completion criteria:

- section-level defaults reduce duplication
- route ownership remains clear

Do not add section metadata if it creates more confusion than value.

## 2.3 Route metadata coverage

Implement route-aware metadata for important public pages.

Completion criteria:

- major public route types are covered
- important pages are not left with generic inherited metadata unless that is truly appropriate

Priority order:

1. homepage
2. key marketing pages
3. blog/article/docs/product/category/detail pages
4. listing/index pages
5. secondary public pages

---

# 3. Static vs dynamic metadata

## 3.1 Use static `metadata` when appropriate

Prefer static `metadata` when:

- the page is stable
- metadata does not depend on route params
- metadata does not depend on CMS/API/entity data
- the page purpose is fixed

Typical candidates:

- homepage
- about
- contact
- fixed landing pages
- stable top-level section pages

Completion criteria:

- stable routes do not use unnecessary dynamic metadata logic

## 3.2 Use `generateMetadata` when appropriate

Prefer `generateMetadata` when metadata depends on:

- slug or route params
- API/CMS/database/headless data
- per-entity title/description/canonical/OG values
- content-specific indexing decisions
- locale-aware alternates that depend on route data

Typical candidates:

- blog posts
- articles
- product detail pages
- category pages with content-driven SEO
- docs detail pages
- author pages
- location/service detail pages

Completion criteria:

- dynamic public pages do not rely on shallow generic metadata
- metadata data source is aligned with page data source

High-risk gap:

- page content is dynamic but metadata remains generic static fallback

---

# 4. Canonical and domain setup

## 4.1 Root domain/base handling

Implement a reliable source for site origin/base URL.

Completion criteria:

- canonical/alternate/OG URLs can be composed safely
- domain handling is centralized enough to avoid drift
- staging/preview leakage risk is reduced

## 4.2 Route canonical coverage

Implement canonical handling where needed for:

- homepage
- major public landing pages
- dynamic detail pages
- listing pages when canonical intent is clear

Completion criteria:

- canonical paths match the intended public URL
- dynamic routes can emit route-correct canonicals
- obvious duplicate-path risk is reduced

Do not:

- point all pages to homepage
- generate canonicals from unstable query params by default

---

# 5. Open Graph and Twitter metadata

## 5.1 Global social defaults

Implement if missing:

- default Open Graph values
- default Twitter values where relevant
- at least one sane default social image strategy

Completion criteria:

- public pages have a baseline social sharing setup

## 5.2 Route-aware social metadata

Implement route-aware social metadata for important public pages when:

- the route title/description is page-specific
- content sharing matters
- public detail pages need meaningful previews

Completion criteria:

- important public detail pages are not stuck with one generic social preview unless intentionally accepted

## 5.3 OG image strategy

Implement one of these:

- static fallback OG image
- route-level static OG image
- generated OG image for dynamic/entity pages

Completion criteria:

- the project has a clear OG image strategy
- social previews are stable and route-appropriate

---

# 6. Robots and indexability foundation

## 6.1 Robots strategy

Implement `robots` when missing or too weak.

Completion criteria:

- robots behavior matches real public/private route intent
- sitemap reference is present when appropriate
- obvious accidental blocking risk is avoided

## 6.2 Indexability intent

Implement or reinforce correct intent for:

- public indexable pages
- internal/private/auth/cart/checkout/account routes
- preview or staging behavior if visible in codebase

Completion criteria:

- implementation does not accidentally expose private routes as SEO targets
- important public pages are not unintentionally blocked

---

# 7. Sitemap foundation

## 7.1 Core sitemap

Implement sitemap support when missing.

Completion criteria:

- sitemap exists
- key public canonical pages are covered
- sitemap strategy matches project size and route model

## 7.2 Dynamic sitemap data

For dynamic content, prefer sitemap generation from:

- API
- CMS
- database
- route-owned data source

Completion criteria:

- major dynamic public entities are not omitted from sitemap without reason

## 7.3 Sitemap scope

Do not include:

- private routes
- auth routes
- utility pages
- non-canonical URLs
- clearly non-indexable pages

Completion criteria:

- sitemap aligns with canonical/indexable intent

---

# 8. Structured data foundation

## 8.1 Add schema only when justified

Implement JSON-LD when:

- route type clearly supports it
- the codebase has enough trustworthy data to populate it correctly
- it adds real SEO value

Typical foundation targets:

- `Organization`
- `WebSite`
- `BreadcrumbList`
- `Article`
- `Product`
- `FAQPage`

Completion criteria:

- schema type matches actual page type
- schema values are derived from real route/page data
- schema is maintainable

## 8.2 Keep schema scoped

Completion criteria:

- avoid one generic schema blob reused across unrelated routes
- keep schema route-appropriate and data-aware

Do not implement speculative or fake schema values.

---

# 9. Rendering and caching alignment

## 9.1 Public stable pages

Prefer:

- static generation when appropriate
- simple metadata paths
- predictable SEO output

Completion criteria:

- stable pages are not needlessly request-time rendered

## 9.2 Public dynamic content pages

Prefer:

- ISR or other freshness-aware strategy when content changes over time
- metadata freshness that matches page data freshness closely enough

Completion criteria:

- public content pages do not use obviously mismatched metadata/data freshness behavior

## 9.3 Request-time rendering

Use only when:

- request context truly matters
- freshness truly requires it
- static or ISR would be meaningfully wrong

Completion criteria:

- request-time rendering is justified, not accidental

Use `skills/shared/RENDERING_AND_CACHING.md` whenever rendering or cache strategy affects SEO behavior.

---

# 10. Images and media essentials

## 10.1 Social images

Implement:

- stable OG/Twitter image paths
- route-aware OG image strategy where appropriate

Completion criteria:

- social previews are not broken or obviously generic where detail pages matter

## 10.2 Content images

Check whether important editorial/product images support:

- meaningful alt text
- stable rendering
- non-broken references

Completion criteria:

- obvious high-value image SEO gaps are not ignored during implementation

Do not drift into full media refactoring unless the gap directly affects SEO quality.

---

# 11. Internal linking and discoverability

## 11.1 Crawlable discovery paths

Check whether important public pages are discoverable through crawlable links from:

- homepage
- listing pages
- blog index
- docs navigation
- category pages
- related content modules
- breadcrumbs if appropriate

Completion criteria:

- key public routes are not orphaned in practice
- detail pages are reachable through normal crawlable navigation

## 11.2 Implement only what belongs to SEO scope

If discoverability is severely weak because templates lack crawlable internal links, implement only the minimal necessary improvements that clearly belong to the SEO foundation.

Do not turn this into a broad UX redesign.

---

# 12. Priority order for implementation

Implement in this order unless the codebase clearly requires another order:

1. routing model and public SEO surface
2. root metadata foundation
3. static vs dynamic metadata decisions
4. canonical and base domain setup
5. route coverage for critical public pages
6. OG/Twitter foundation
7. robots
8. sitemap
9. structured data foundation
10. rendering/caching alignment
11. internal linking/discoverability improvements
12. image/media SEO refinements

---

# 13. Completion test

Implementation is considered solid when most of these are true:

- the app has one clear metadata foundation
- important public routes have route-appropriate metadata
- dynamic content pages use dynamic metadata when needed
- canonical behavior is coherent
- domain/base URL handling is safe
- OG/Twitter setup exists and is not purely accidental
- robots and sitemap exist and reflect real SEO intent
- schema exists where clearly beneficial and supportable
- rendering strategy does not obviously undermine SEO output
- the implementation is maintainable and not scattered

---

# 14. Stop conditions

Do not keep expanding implementation once:

- the essential SEO foundation is in place
- important public route types are covered
- remaining work becomes content strategy, editorial work, or deep product/domain logic
- further changes would mainly be cleanup/refactor rather than implementation

At that point:

- finish the implementation
- report remaining gaps clearly
- leave advanced correction work for `seo-fix`

---

# 15. Output discipline

When using this checklist, the final result should clearly state:

- what foundation was implemented
- which route types are now covered
- what was intentionally not implemented
- what still requires `seo-fix` or later refinement
