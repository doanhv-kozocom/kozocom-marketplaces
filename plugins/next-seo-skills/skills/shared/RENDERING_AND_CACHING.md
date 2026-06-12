Use this file to audit rendering strategy, caching, and revalidation for SEO-critical pages in Next.js.

Purpose:

- choose the correct rendering mode for public SEO pages
- keep HTML output stable enough for crawlability and shareability
- align metadata freshness with page-content freshness
- avoid accidental dynamic rendering, accidental no-store, or mismatched revalidation
- make recommendations that fit both Next.js docs and the project’s existing architecture

Priority order:

1. Official Next.js docs
2. If available locally, Vercel skill for platform-aware rendering and caching validation
3. Existing project architecture and codebase evidence
4. Fallback reasoning only when the above are insufficient

General rules:

- Public SEO pages should use the simplest rendering strategy that satisfies freshness requirements
- Shared public content should prefer deterministic output
- Do not recommend request-time rendering unless the business requirement clearly justifies it
- Evaluate page content, metadata, sitemap freshness, and canonical stability together
- Prefer the project’s current conventions unless they are clearly harming SEO, consistency, or performance
- For App Router, first determine whether the project uses the current Cache Components model or the previous caching model

---

1. First identify the caching model

Reference:
https://nextjs.org/docs/app/getting-started/caching
https://nextjs.org/docs/app/getting-started/revalidating
https://nextjs.org/docs/app/api-reference/config/next-config-js/cacheComponents
https://nextjs.org/docs/app/guides/caching-without-cache-components

Pattern:

- Newer App Router docs describe caching and revalidation through Cache Components
- Projects not using that model should be evaluated with the previous caching model docs
- Do not mix guidance from both models without checking what the codebase is actually doing

Audit steps:

- check whether `cacheComponents` is enabled
- check whether the project uses `use cache`
- check whether the project relies on `fetch(..., { next: { revalidate } })`, `cache: 'force-cache'`, `cache: 'no-store'`, or route segment config
- check whether the project uses older cache patterns consistently across the route tree

Do:

- state explicitly which caching model appears to be in use
- keep recommendations compatible with that model

Do not:

- recommend a pattern from one model when the codebase clearly uses the other
- assume the latest caching model if the project is built on the previous model

---

2. Strategy order for public SEO pages

Reference:
https://nextjs.org/docs/app/guides/public-static-pages
https://nextjs.org/docs/app/guides/incremental-static-regeneration
https://nextjs.org/docs/app/getting-started/caching
https://vercel.com/docs/frameworks/full-stack/nextjs

Default preference for public SEO pages:

1. SSG
2. ISR
3. SSR

Use this order unless content freshness or request-specific behavior clearly requires otherwise.

Reasoning:

- Stable prerendered HTML is usually the best starting point for crawlability, consistency, and performance
- ISR is usually the best upgrade when public content changes over time
- SSR should be reserved for content that genuinely must be rendered per request

Do:

- prefer static output for public shared pages
- prefer ISR over SSR when timed freshness is enough
- explain why a more dynamic strategy is necessary before recommending it

Do not:

- default to SSR for public content
- recommend full request-time rendering for public SEO pages without strong evidence

---

3. SSG evaluation

Reference:
https://nextjs.org/docs/app/guides/public-static-pages
https://nextjs.org/docs/app/api-reference/functions/generate-static-params

Prefer SSG for:

- homepage when mostly stable
- marketing pages
- landing pages
- about, contact, pricing, policy pages
- stable docs pages
- stable category or overview pages
- public content that changes rarely

Signals that SSG is a strong fit:

- content is public
- output is shared across users
- content is stable across requests
- freshness is not critical on every request
- slugs are known or the page is not route-dynamic

Audit checks:

- is the route public
- is the HTML stable across visitors
- does the page depend on request headers, auth, geo, cookies, or other request-only state
- is the page needlessly rendered dynamically

Do:

- recommend SSG for stable pages with low freshness pressure
- recommend static metadata for truly stable pages

Do not:

- recommend SSR when no request-time data is actually required
- classify a page as static if its visible output truly depends on request-only state

---

4. generateStaticParams and route prerendering

Reference:
https://nextjs.org/docs/app/api-reference/functions/generate-static-params

Pattern:

- use `generateStaticParams` when dynamic public routes can be enumerated ahead of time
- it is especially useful for blog, docs, category, and product routes where the route set is known at build time
- `generateStaticParams` runs at build time and is not called again during ISR

Implications for audit:

- do not rely on `generateStaticParams` as the mechanism that refreshes route inventory after deploy
- if the content source changes frequently, assess whether the route strategy accounts for newly added slugs
- separate “route enumeration” from “page revalidation”

Do:

- recommend `generateStaticParams` when important public routes can be prerendered safely
- point out when a site expects new content but has no clear plan for new slugs

Do not:

- assume ISR will rerun `generateStaticParams`
- treat prerendered route params as a substitute for content revalidation

---

5. ISR evaluation

Reference:
https://nextjs.org/docs/app/guides/incremental-static-regeneration
https://nextjs.org/docs/app/getting-started/revalidating
https://nextjs.org/docs/app/guides/caching-without-cache-components
https://vercel.com/docs/incremental-static-regeneration

Prefer ISR for:

- blog pages
- article pages
- docs pages that update after deploy
- product and category pages that change periodically
- CMS-backed or API-backed public content
- public pages where freshness matters but not on every request

Signals that ISR is a strong fit:

- content is public and shared across users
- HTML should remain stable for bots and users between updates
- updates happen on a schedule of minutes or hours, not every request
- the site benefits from serving cached HTML while refreshing in the background

Audit checks:

- are routes using revalidation intentionally
- is revalidation interval aligned with business freshness needs
- is ISR used where the page is public and content updates periodically
- is the project incorrectly using SSR where ISR would satisfy freshness

Do:

- recommend ISR for content-driven public pages when freshness is periodic
- recommend matching content revalidation and metadata revalidation strategy
- recommend production-style testing for ISR behavior

Do not:

- recommend fully dynamic rendering when time-based or on-demand revalidation is enough
- set arbitrary revalidation values without a freshness reason

---

6. SSR and request-time rendering evaluation

Reference:
https://nextjs.org/docs/app/getting-started/caching
https://nextjs.org/docs/app/getting-started/fetching-data
https://vercel.com/docs/frameworks/full-stack/nextjs

Prefer SSR or request-time rendering only when:

- output must be fresh on every request
- output depends on request context
- output depends on authentication, cookies, headers, geo, or user-specific conditions
- serving cached shared HTML would be incorrect

SEO-specific guidance:

- public SEO landing pages, blog posts, docs pages, and most catalog pages should not default to SSR
- SSR is appropriate when the public output truly changes per request or must always reflect live state
- if recommending SSR, explain why SSG or ISR is insufficient

Audit checks:

- does the page use request-only APIs
- does the page depend on user-specific state
- is `no-store` used intentionally
- is SSR harming performance or consistency on a page that could be static or ISR

Do:

- recommend SSR only when request-time correctness clearly outweighs caching benefits
- note when public content is made dynamic without a business reason

Do not:

- recommend SSR as a generic “SEO-safe” default
- keep public shared pages dynamic if their data model does not require it

---

7. Metadata freshness alignment

Reference:
https://nextjs.org/docs/app/api-reference/functions/generate-metadata
https://nextjs.org/docs/app/getting-started/revalidating
https://nextjs.org/docs/app/guides/incremental-static-regeneration

Pattern:

- metadata freshness should align with page-content freshness
- if page content is fetched from API, CMS, or database, metadata should usually be derived from the same source
- if the page uses revalidation, metadata should not remain generic or stale for route-specific content

Strong expectation:

- blog, article, product, category, docs, and slug pages should usually use `generateMetadata`
- metadata for those pages should prefer the same source of truth as the page body

Negative signals:

- page content is dynamic or ISR-based but metadata is generic
- metadata is static while route content changes by slug
- canonical, OG, or Twitter data is disconnected from page content
- page content and metadata appear to use different freshness models without reason

Do:

- recommend aligning metadata and page-data strategy
- recommend `generateMetadata` for data-driven routes

Do not:

- accept generic root metadata as sufficient for SEO-critical detail pages
- accept stale metadata on routes whose content is clearly dynamic

---

8. Cache evaluation

Reference:
https://nextjs.org/docs/app/getting-started/caching
https://nextjs.org/docs/app/guides/caching-without-cache-components
https://nextjs.org/docs/app/api-reference/directives/use-cache

Pattern:

- stable public content should not be accidentally uncached
- public pages should not use `no-store` without a strong reason
- caching decisions should match the route’s freshness requirements
- cache directives should be intentional, consistent, and explainable

Audit checks:

- whether stable public content is using `cache: 'no-store'`
- whether `revalidate` exists where content changes periodically
- whether fetch caching choices across a route are coherent
- whether the project’s cache behavior makes the route more dynamic than necessary
- whether cache policy is stable across layout and page composition

Do:

- prefer stable caching for shared public content
- prefer explicit revalidation for periodically updated content
- prefer narrow dynamic behavior instead of full-route dynamism where possible

Do not:

- recommend `no-store` by default
- allow one fetch decision to make the whole route more dynamic without understanding the impact

---

9. Revalidation evaluation

Reference:
https://nextjs.org/docs/app/getting-started/revalidating
https://nextjs.org/docs/app/guides/caching-without-cache-components

Pattern:

- revalidation is for keeping cached data fresh while serving fast responses
- the revalidation strategy should reflect the actual update pattern of the content
- route-level and fetch-level revalidation need to be understood together

Audit checks:

- are revalidation values present where public content changes
- are they too aggressive or too lax for the business requirement
- is time-based revalidation enough or is on-demand revalidation more suitable
- are different data sources in the same page using conflicting freshness assumptions

Do:

- recommend revalidation windows based on content change frequency
- call out when “always fresh” is being used where “fresh enough” would work

Do not:

- recommend arbitrary numbers without linking them to content update behavior
- mix incompatible caching assumptions inside the same route without explanation

---

10. CDN and deployment-aware caching

Reference:
https://nextjs.org/docs/app/guides/cdn-caching
https://vercel.com/docs/caching/runtime-cache/data-cache
https://vercel.com/docs/frameworks/full-stack/nextjs

Pattern:

- deployment behavior matters for SEO because caching affects HTML stability, latency, and freshness
- on Vercel, App Router deployments can use Data Cache alongside ISR
- platform-aware analysis is useful when the project relies heavily on caching or dynamic data

If a local Vercel skill is available:

- use it to validate rendering mode
- use it to validate cache and revalidation assumptions
- use it to check whether the current behavior matches what the platform will actually do in production

If no Vercel skill is available:

- evaluate from Next.js code patterns only
- state uncertainty where production platform behavior cannot be verified from code alone

Do:

- consider hosting/runtime behavior when auditing SEO-critical public routes
- use Vercel skill as a validation layer, not a replacement for reading the codebase

Do not:

- assume local dev behavior fully reflects production caching
- make deployment-specific claims without code or platform evidence

---

11. Production validation guidance

Reference:
https://nextjs.org/docs/app/guides/incremental-static-regeneration

Pattern:

- ISR and cache behavior should be validated in production-like mode
- local development is not enough to judge final caching behavior

Audit guidance:

- when recommending ISR fixes, suggest validating with `next build` and `next start`
- if the project uses Vercel, platform-aware checks are strongly useful
- if needed, recommend debug steps for cache verification

Do:

- distinguish between dev-mode behavior and production-mode behavior
- call out when a rendering recommendation needs production validation

Do not:

- treat dev-mode observations as sufficient proof of final cache behavior

---

12. Codebase-first recommendation rules

Before recommending rendering or caching changes:

- check whether the project already uses `generateStaticParams`
- check whether the project already uses `revalidate`
- check whether fetch calls already define caching behavior
- check whether route-level behavior is already consistent
- check whether metadata strategy matches page-data strategy
- check whether the current render mode appears intentional

Do:

- recommend the smallest correct change
- preserve project conventions where possible
- explain the SEO reason and the runtime reason together

Do not:

- change render strategy without clear benefit
- recommend architectural rewrites when a local fix is enough

---

13. Quick decision matrix

Use this matrix when recommending a direction:

Choose SSG when:

- page is public
- content is stable
- output is shared across users
- freshness pressure is low

Choose ISR when:

- page is public
- content changes over time
- freshness matters but not on every request
- stable cached HTML is still desirable

Choose SSR when:

- output must be request-time fresh
- output depends on request context
- cached shared output would be incorrect

Escalate review when:

- metadata strategy does not match page-data strategy
- route inventory changes often but static params are the only route mechanism
- caching behavior cannot be confidently inferred from the codebase
- production hosting behavior is important and should be checked with Vercel skill

---

14. Anti-patterns

Treat these as strong negative signals:

- public SEO pages fully dynamic without a clear reason
- stable public pages using `no-store`
- data-driven public content with no clear revalidation strategy
- metadata freshness disconnected from page-content freshness
- `generateStaticParams` implicitly relied on as a refresh mechanism
- route-level caching behavior that appears accidental rather than intentional
- recommendations that ignore the project’s existing architecture
