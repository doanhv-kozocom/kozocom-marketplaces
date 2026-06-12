# CHECKLIST.md

This file is the decision backbone for `seo-plan`.

Its purpose is to:

- inspect the current SEO state of a Next.js project
- distinguish **missing foundation** from **broken existing implementation**
- decide whether the next step should be `seo-implement` or `seo-fix`

Use this checklist strictly.
Do not skip sections just because a few metadata fields already exist.

---

# 1 Project classification

## 1.1 Framework and router

Check:

- Is this a Next.js project?
- Is it using `app/` (App Router)?
- Is it using `pages/` only?
- Is it hybrid?

Mark:

- `PASS` = clearly identified
- `RISK` = mixed or non-standard setup
- `FAIL` = framework/router cannot be determined from source

## 1.2 Public SEO surface

Identify all public, indexable, SEO-relevant route groups:

- homepage
- marketing pages
- blog/news/article pages
- docs/help pages
- category/listing pages
- product/detail pages
- location/service pages
- author/tag pages
- search/filter pages if public
- localized versions if any

Mark:

- `PASS` = public SEO surface is mapped
- `RISK` = some public routes are unclear
- `FAIL` = important public routes are not identifiable

## 1.3 Non-indexable surface

Identify routes that likely should not be indexed:

- login/register
- account/dashboard
- cart/checkout
- internal tools
- preview/staging/debug pages
- private user pages
- parameterized utility pages

Mark:

- `PASS` = clearly separated from public SEO surface
- `RISK` = likely mixed with public routes
- `FAIL` = no distinction can be inferred

---

# 2 Metadata architecture

## 2.1 Global metadata foundation

Check whether the project has a clear global metadata layer in the correct root layout scope.

Look for:

- root `layout.tsx|js`
- global title strategy
- default description strategy
- `metadataBase`
- robots defaults
- default OG/Twitter defaults
- icons

Mark:

- `PASS` = clear global metadata foundation exists
- `RISK` = partial or weak global foundation
- `FAIL` = little or no global metadata foundation

## 2.2 Route-level metadata coverage

For every important public route type, check whether metadata is defined at the correct page/layout level.

Ask:

- Do key route types have route-aware metadata?
- Are important page types missing dedicated metadata?
- Are many pages inheriting generic metadata that should be specific?

Mark:

- `PASS` = major route types have proper coverage
- `RISK` = partial coverage or overly generic coverage
- `FAIL` = public route metadata is mostly absent

## 2.3 Static metadata vs dynamic metadata

This is a key decision point.

Use **static `metadata`** when page metadata is stable and does not need entity-specific or request-specific values.

Prefer **`generateMetadata`** when metadata depends on:

- route params
- slug/entity data
- CMS/API/headless content
- per-page canonical/title/description/OG values
- dynamic alternates
- data-driven indexability decisions

If a page is a dynamic content page but only uses generic static metadata, treat it as a gap.

Mark:

- `PASS` = static/dynamic choice matches the route’s data nature
- `RISK` = some dynamic routes use shallow static metadata
- `FAIL` = metadata strategy is fundamentally mismatched

## 2.4 Metadata inheritance and duplication

Check:

- nested layout overrides
- duplicated title logic
- conflicting metadata ownership
- generic metadata leaking into detail pages
- multiple competing SEO utilities

Mark:

- `PASS` = ownership is clear and consistent
- `RISK` = some duplication or unclear ownership
- `FAIL` = conflicting or fragmented metadata architecture

## 2.5 Canonical readiness

Check:

- canonical support exists where needed
- canonical domain is correct
- dynamic pages can produce route-correct canonicals
- query-param or duplicate-path issues are accounted for

Mark:

- `PASS` = canonical strategy is visible and coherent
- `RISK` = partial or weak canonical handling
- `FAIL` = missing or likely incorrect canonical strategy

---

# 3 Metadata file conventions

## 3.1 Robots

Check:

- presence of `robots.txt` or generated robots config
- correct allow/disallow intent
- sitemap reference
- no accidental blocking of important public routes

Mark:

- `PASS` = robots exists and intent is coherent
- `RISK` = robots exists but may be incomplete or risky
- `FAIL` = robots is missing or clearly unsafe

## 3.2 Sitemap

Check:

- sitemap exists
- key public routes appear representable
- dynamic routes are covered
- large sites can scale to split sitemaps if needed

Mark:

- `PASS` = sitemap coverage strategy exists
- `RISK` = sitemap exists but appears incomplete
- `FAIL` = no sitemap strategy exists

## 3.3 Icons and brand assets

Check:

- favicon/icon/apple-icon coverage
- assets are not missing or placeholder-grade for production

Mark:

- `PASS` = icon coverage exists
- `RISK` = incomplete or weak brand asset setup
- `FAIL` = missing

## 3.4 OG image strategy

Check:

- default OG image exists
- route-level OG image strategy exists where appropriate
- dynamic content pages can produce content-relevant OG images if needed

Mark:

- `PASS` = OG image strategy is present and usable
- `RISK` = only generic fallback exists
- `FAIL` = OG image handling is absent

---

# 4 Indexability and crawl control

## 4.1 Intended indexable pages

Check whether the project clearly implies which pages should be indexable.

Mark:

- `PASS` = indexable targets are clear
- `RISK` = some route types are ambiguous
- `FAIL` = no clear indexability model

## 4.2 Accidental noindex / accidental index

Check:

- pages that should rank but may be blocked
- pages that should not rank but appear indexable
- environment-specific leakage

Mark:

- `PASS` = intent appears controlled
- `RISK` = some suspicious cases
- `FAIL` = strong signs of wrong indexing behavior

## 4.3 Duplicate URL risk

Check for likely duplicate indexable variants:

- query params
- filtered URLs
- pagination variants
- trailing slash or route alias duplication
- locale duplication
- staging/production duplication

Mark:

- `PASS` = duplication risk appears addressed
- `RISK` = some duplication risks visible
- `FAIL` = major duplication risk is likely unhandled

---

# 5 Structured data

## 5.1 Coverage by page type

Check whether structured data exists where it reasonably should, such as:

- Organization / WebSite
- BreadcrumbList
- Article / BlogPosting
- Product
- FAQPage
- LocalBusiness

Mark:

- `PASS` = relevant schema coverage exists
- `RISK` = partial schema coverage
- `FAIL` = schema is broadly absent where it matters

## 5.2 Correctness and placement

Check:

- schema type fits page type
- schema is not obviously duplicated
- fields are not clearly misleading or stale
- injection point is maintainable

Mark:

- `PASS` = schema looks coherent
- `RISK` = weak or partial quality
- `FAIL` = schema appears wrong, duplicated, or fragile

---

# 6 Rendering, data fetching, and caching impact

## 6.1 SEO-critical routes vs rendering mode

Check whether important public pages are:

- static where reasonable
- dynamic only when justified
- revalidated sensibly if content changes over time

Mark:

- `PASS` = rendering choice appears reasonable
- `RISK` = some SEO-important routes look suboptimal
- `FAIL` = rendering choice likely harms SEO quality or stability

## 6.2 Metadata data source behavior

Check whether metadata relies on:

- CMS/API data
- unstable runtime-only logic
- request-time values without strong reason
- duplicated data fetching patterns

Mark:

- `PASS` = metadata data flow appears intentional
- `RISK` = unnecessary complexity or fragility
- `FAIL` = metadata flow is likely wrong or brittle

## 6.3 Staleness and cache risk

Check for likely SEO staleness issues:

- metadata may not refresh when content changes
- canonical/OG/title may lag behind entity updates
- dynamic pages are forced static incorrectly
- caching rules are unclear around metadata generation

Mark:

- `PASS` = staleness risk appears controlled
- `RISK` = some possible stale SEO behavior
- `FAIL` = strong stale metadata risk

---

# 7 Content-template signals from source

## 7.1 Title and description uniqueness potential

Check whether templates support meaningful uniqueness for major public page types.

Mark:

- `PASS` = route templates can produce specific metadata
- `RISK` = many routes likely share generic metadata
- `FAIL` = uniqueness appears fundamentally weak

## 7.2 Heading and page-purpose clarity

Check source templates for:

- one clear page purpose
- sensible heading structure
- enough template specificity for indexable pages

Mark:

- `PASS` = templates support clear topical pages
- `RISK` = some template-level ambiguity
- `FAIL` = templates are too generic/thin for SEO-critical pages

## 7.3 Internal linking signals

Check visible template/component patterns for:

- breadcrumb links
- related links
- list-to-detail links
- weak generic anchors everywhere

Mark:

- `PASS` = internal linking signals exist in templates
- `RISK` = limited linking signals
- `FAIL` = little useful internal linking signal is visible

---

# 8 International SEO (only if applicable)

## 8.1 Locale architecture

Check:

- locale paths/subpaths/subdomains if visible
- default locale behavior
- route duplication risk across locales

Mark:

- `PASS` = locale architecture is understandable
- `RISK` = multilingual setup exists but is unclear
- `FAIL` = multilingual setup appears broken or duplicated

## 8.2 Alternates / hreflang readiness

Check whether locale-aware alternates are possible and likely implemented correctly.

Mark:

- `PASS` = alternate strategy appears coherent
- `RISK` = partial locale alternate handling
- `FAIL` = multilingual routes exist without clear alternate strategy

---

# 9 Environment and domain correctness

## 9.1 Domain consistency

Check:

- canonical domain source
- `metadataBase`
- OG URL domain
- sitemap domain
- robots sitemap reference
- staging/preview leakage

Mark:

- `PASS` = domain behavior appears production-correct
- `RISK` = some domain handling is unclear
- `FAIL` = strong sign of wrong domain output

## 9.2 Deployment-awareness

Check whether SEO-critical values appear hardcoded unsafely or environment-dependent in fragile ways.

Mark:

- `PASS` = deployment-aware setup looks safe
- `RISK` = some env fragility
- `FAIL` = obvious production-risky setup

---

# 10 Scoring logic

Use this simple scoring model:

- `PASS` = 2
- `RISK` = 1
- `FAIL` = 0

Do not obsess over exact math.
Use scoring to support judgment, not replace judgment.

## Critical sections

These sections have highest weight:

- 2. Metadata architecture
- 3. Metadata file conventions
- 4. Indexability and crawl control
- 6. Rendering, data fetching, and caching impact
- 9. Environment and domain correctness

If these are weak, the project likely needs stronger intervention even if minor SEO pieces exist.

---

# 11 Decision rules: seo-implement vs seo-fix

## Recommend `seo-implement` when:

- global metadata foundation is missing or very weak
- route-level metadata coverage is mostly absent
- dynamic content pages do not have proper metadata strategy
- robots/sitemap/OG/schema are mostly absent
- SEO exists only as scattered basic tags
- public SEO architecture is not really established

In short:

- there is **not enough SEO system to repair**
- the project needs a proper baseline implementation

## Recommend `seo-fix` when:

- a recognizable SEO system already exists
- metadata is already implemented in multiple relevant areas
- robots/sitemap/schema/canonical logic already exist at least partially
- the main issues are wrong values, wrong ownership, missing coverage, conflicts, duplication, fragility, or weak scalability

In short:

- the system exists
- it needs correction and strengthening, not first-time setup

## Mixed case rule

If part of the project has good SEO and another important public section has almost none:

- choose the dominant recommendation for the overall public surface
- explicitly call out the mixed state

Example:

- “Recommend seo-fix overall, but implement missing SEO foundation for the blog module.”

---

# 12 Hard rules for the final decision

Choose `seo-implement` if **3 or more** of these are true:

- no meaningful global metadata foundation
- no solid route-level metadata strategy
- no real dynamic metadata for dynamic content
- no usable robots/sitemap setup
- no meaningful schema strategy
- no coherent canonical/domain strategy

Choose `seo-fix` if **3 or more** of these are true:

- metadata architecture already exists
- route-level SEO exists in key public areas
- robots or sitemap already exists
- schema already exists in at least some important pages
- canonical/domain handling already exists
- the main issues are correction, cleanup, or extension

If still unclear:

- prefer the recommendation that best fits the **majority of SEO-critical public pages**
- do not let one well-optimized route hide a weak overall system

---

# 13 Output discipline

The final audit must:

- separate verified facts from inference
- separate foundational gaps from corrective gaps
- prioritize by impact
- end with one explicit recommendation only:
  - `seo-implement`
  - `seo-fix`

Never return an ambiguous ending such as:

- “either could work”
- “depends on preference”
- “both are valid”

You must decide.
