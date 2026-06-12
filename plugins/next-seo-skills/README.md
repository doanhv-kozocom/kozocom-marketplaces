# Next.js SEO Skills

A focused Claude Code skill set for planning, implementing, and fixing SEO in Next.js projects.

## Skills

### `seo-plan`

Reviews the project, checks current SEO maturity, identifies gaps, and decides the next step:

- `seo-implement`
- `seo-fix`

### `seo-implement`

Builds SEO foundations for projects with little or no real SEO.

Typical scope:

- metadata
- `generateMetadata`
- canonical handling
- Open Graph / Twitter
- robots
- sitemap
- structured data
- SEO-aware rendering

### `seo-fix`

Fixes and improves existing SEO implementation without rebuilding everything from scratch.

Typical scope:

- incorrect or generic metadata
- weak dynamic metadata
- broken canonicals
- inconsistent robots/sitemap
- weak OG/Twitter setup
- fragile schema
- rendering/cache issues affecting SEO

## Recommended flow

1. Run `seo-plan`
2. Continue with:
   - `seo-implement` for missing SEO foundations
   - `seo-fix` for existing but flawed SEO

## Structure

```txt
skills/
  shared/
    SNIPPETS.md
    RENDERING_AND_CACHING.md

  seo-plan/
    SKILL.md
    CHECKLIST.md

  seo-implement/
    SKILL.md
    CHECKLIST.md

  seo-fix/
    SKILL.md
    CHECKLIST.md
```

## Shared files

### `shared/SNIPPETS.md`

Version-aware Next.js SEO reference patterns based on official docs.

### `shared/RENDERING_AND_CACHING.md`

Guidance for rendering and caching decisions that affect SEO.

## Principles

- Prefer official Next.js patterns
- Prefer route-aware SEO ownership
- Prefer dynamic metadata for dynamic content
- Treat SEO as a system, not just tags
- Keep implementations maintainable and codebase-aware
