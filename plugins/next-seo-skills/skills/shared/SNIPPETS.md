This file is the version-aware reference library for `seo-plan`, `seo-implement`, and `seo-fix`.

Its purpose is to:

- provide **framework-native Next.js SEO patterns**
- prefer **official Next.js documentation patterns**
- choose snippets based on the **actual Next.js version in the user's codebase**
- prevent outdated or cross-version-misaligned SEO implementations

This file is not a generic SEO article.
It is a practical snippet reference.

---

# 1. Version resolution rules

## 1.1 Detect the project version first

Before using any snippet, inspect:

- `package.json`
- lockfile if needed
- installed `next` version if visible

Extract:

- exact `next` version
- major version

## 1.2 Documentation priority

Use snippets in this order:

1. **Exact major-version docs** matching the project
2. **Latest docs** only when:
   - the pattern is unchanged
   - or version-specific docs are not necessary
3. Existing project conventions only if they are still compatible with official Next.js behavior

## 1.3 Canonical doc mapping

Use this mapping when referencing documentation:

- If project uses **Next 15**:
  - prefer docs under:
    - `https://nextjs.org/docs/15/...`
  - especially:
    - `https://nextjs.org/docs/15/app/getting-started/metadata-and-og-images`

- If project uses the **current/latest major**:
  - prefer docs under:
    - `https://nextjs.org/docs/...`
  - especially:
    - `https://nextjs.org/docs/app/getting-started/metadata-and-og-images`

## 1.4 Conflict rule

If a snippet from memory, an old blog post, or existing project code conflicts with official Next.js docs:

- prefer the official docs pattern
- unless the current project version clearly requires an older compatible pattern

---

# 2. Snippet selection rules

## 2.1 Prefer static `metadata` when metadata is stable

Use `export const metadata` when:

- page metadata is stable
- route does not depend on slug/entity data
- route does not need CMS/API-derived title/description
- route can safely share predictable metadata at build time

Typical fit:

- homepage
- about
- contact
- fixed marketing pages
- static docs landing pages

## 2.2 Prefer `generateMetadata` when metadata depends on data

Use `generateMetadata` when metadata depends on:

- dynamic route params
- slug/entity data
- API/CMS/headless content
- per-item title/description/canonical/OG data
- route-specific alternates
- page data that cannot be expressed safely as one static metadata object

Typical fit:

- blog post pages
- article detail pages
- product detail pages
- category pages with server-side data-driven metadata
- author pages
- location/service detail pages

Hard rule:
If the page is a dynamic public page and the metadata must reflect entity data, do **not** keep generic static metadata just because the route technically renders.

## 2.3 Prefer metadata file conventions where Next.js already provides them

Use file conventions instead of custom ad-hoc handling where appropriate:

- `opengraph-image.*`
- `twitter-image.*`
- `robots.txt` / `robots.ts`
- `sitemap.xml` / `sitemap.ts`
- `favicon.ico`
- `icon.*`
- `apple-icon.*`

## 2.4 Prefer `metadataBase` at the root

If canonical URLs, alternate URLs, or OG image paths use relative paths, prefer configuring `metadataBase` in the root layout.

Hard rule:
Do not build SEO URLs from scattered string concatenation if `metadataBase` can solve it cleanly.

## 2.5 Prefer route-native ownership

Put metadata where route ownership is clearest:

- global defaults in root layout
- section-level defaults in nested layout
- entity-specific metadata in route page or route-owned logic

Avoid:

- one giant SEO utility controlling unrelated routes without clear ownership
- duplicated metadata logic across multiple files without hierarchy

---

# 3. Core snippets

## 3.1 Static metadata for stable routes

Use when:

- route is stable
- metadata is not entity-driven
- no API/CMS lookup is needed

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "About",
  description: "Learn more about our company and services.",
};

export default function Page() {
  return <main>...</main>;
}
```

Use this for:

- simple marketing pages
- stable informational pages

Do not use this pattern for:

- blog `[slug]`
- product `[id]`
- CMS-driven article detail pages
- any page where title/description should come from fetched content

---

## 3.2 Dynamic metadata for slug/entity pages

Use when:

- metadata depends on route params
- data comes from API/CMS/headless backend
- canonical/title/description/OG should reflect the entity

```tsx
import type { Metadata, ResolvingMetadata } from "next";

type Props = {
  params: Promise<{ slug: string }>;
};

async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`);
  if (!res.ok) throw new Error("Failed to fetch post");
  return res.json();
}

export async function generateMetadata(
  { params }: Props,
  parent: ResolvingMetadata,
): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);

  return {
    title: post.seoTitle || post.title,
    description: post.seoDescription || post.excerpt,
  };
}

export default async function Page({ params }: Props) {
  const { slug } = await params;
  const post = await getPost(slug);

  return <article>{post.title}</article>;
}
```

Prefer this over static metadata when:

- the page is public and dynamic
- SEO values are content-driven
- the content source is external or headless

---

## 3.3 Root metadata with `metadataBase`

Use when:

- the app needs canonical URLs
- alternates use relative paths
- OG image URLs use relative paths
- the domain should be centrally controlled

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  metadataBase: new URL("https://example.com"),
  title: {
    default: "Example",
    template: "%s | Example",
  },
  alternates: {
    canonical: "/",
  },
  openGraph: {
    images: "/og-image.png",
  },
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

Prefer this pattern over:

- hardcoding full canonical URLs separately in many pages
- mixing environment/domain logic throughout route files

---

## 3.4 Route-level canonical with root `metadataBase`

Use when:

- each route needs its own canonical path
- the domain is centralized in root layout

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  alternates: {
    canonical: "/about",
  },
};

export default function Page() {
  return <main>About</main>;
}
```

This is preferred when:

- the route path is fixed
- the root layout already defines `metadataBase`

---

## 3.5 Dynamic canonical for entity pages

Use when:

- canonical depends on slug/id
- route is dynamic and public

```tsx
import type { Metadata } from "next";

type Props = {
  params: Promise<{ slug: string }>;
};

async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`);
  if (!res.ok) throw new Error("Failed to fetch post");
  return res.json();
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);

  return {
    title: post.seoTitle || post.title,
    description: post.seoDescription || post.excerpt,
    alternates: {
      canonical: `/blog/${slug}`,
    },
  };
}
```

Use this when:

- the canonical must match the actual entity URL
- the page is backed by dynamic data

---

# 4. Open Graph and social snippets

## 4.1 Static OG image for app-wide fallback

Use when:

- one default OG image is acceptable for most routes
- social sharing does not need per-entity image generation

File:

```txt
app/opengraph-image.jpg
```

Optional route-specific static file:

```txt
app/blog/opengraph-image.jpg
```

Prefer this when:

- a simple branded fallback is enough
- there is no need for per-entity share cards yet

## 4.2 Generated OG image for entity pages

Use when:

- each post/product/page should have a unique social image
- the OG image depends on dynamic content

```tsx
import { ImageResponse } from "next/og";

export const size = {
  width: 1200,
  height: 630,
};

export const contentType = "image/png";

export default async function Image() {
  return new ImageResponse(
    <div
      style={{
        fontSize: 64,
        background: "white",
        width: "100%",
        height: "100%",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
      }}
    >
      Example Post Title
    </div>,
    {
      ...size,
    },
  );
}
```

Typical location:

```txt
app/blog/[slug]/opengraph-image.tsx
```

Prefer generated OG images when:

- article/product/detail pages are major sharing surfaces
- branded previews matter
- the project already has entity data available on the server

---

# 5. Robots snippets

## 5.1 Static robots file

Use when:

- rules are simple
- environment logic is not needed

```txt
User-Agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
```

Use a stricter variant when needed:

```txt
User-Agent: *
Allow: /
Disallow: /account/
Disallow: /checkout/

Sitemap: https://example.com/sitemap.xml
```

## 5.2 Programmatic robots

Use when:

- rules vary by environment
- sitemap or host needs centralized generation
- you want typed, maintainable config

```ts
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: "*",
      allow: "/",
      disallow: ["/account/", "/checkout/"],
    },
    sitemap: "https://example.com/sitemap.xml",
  };
}
```

Prefer programmatic robots when:

- deployment environment matters
- private/public route separation needs maintenance

---

# 6. Sitemap snippets

## 6.1 Static sitemap for small stable sites

Use when:

- site is small
- route list is stable
- sitemap does not need data fetching

```xml
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com</loc>
  </url>
  <url>
    <loc>https://example.com/about</loc>
  </url>
</urlset>
```

## 6.2 Programmatic sitemap for real projects

Use when:

- routes are generated
- dynamic content must be included
- maintainability matters

```ts
import type { MetadataRoute } from "next";

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: "https://example.com",
      lastModified: new Date(),
      changeFrequency: "weekly",
      priority: 1,
    },
    {
      url: "https://example.com/about",
      lastModified: new Date(),
      changeFrequency: "monthly",
      priority: 0.8,
    },
  ];
}
```

Prefer this pattern when:

- the app has dynamic content
- sitemap should be generated from code rather than manually maintained XML

## 6.3 Dynamic sitemap for CMS/headless/blog/product data

Use when:

- the site has many dynamic pages
- content comes from database/CMS/API

```ts
import type { MetadataRoute } from "next";

async function getPosts() {
  const res = await fetch("https://api.example.com/posts");
  if (!res.ok) throw new Error("Failed to fetch posts");
  return res.json();
}

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getPosts();

  return [
    {
      url: "https://example.com",
      lastModified: new Date(),
      priority: 1,
    },
    ...posts.map((post: { slug: string; updatedAt: string }) => ({
      url: `https://example.com/blog/${post.slug}`,
      lastModified: new Date(post.updatedAt),
      changeFrequency: "weekly" as const,
      priority: 0.7,
    })),
  ];
}
```

## 6.4 Multiple sitemaps for large sites

Use when:

- the site is large
- one sitemap is not enough
- route groups or entity volume justify splitting

```ts
import type { MetadataRoute } from "next";

export async function generateSitemaps() {
  return [{ id: 0 }, { id: 1 }];
}

export default async function sitemap(props: {
  id: Promise<string>;
}): Promise<MetadataRoute.Sitemap> {
  const id = Number(await props.id);

  const start = id * 50000;
  const end = start + 50000;

  return Array.from({ length: Math.max(end - start, 0) })
    .slice(0, 2)
    .map((_, i) => ({
      url: `https://example.com/product/${start + i}`,
      lastModified: new Date(),
    }));
}
```

Prefer this when:

- the project is large enough to need segmented sitemaps
- product/article/directory listings are high-volume

---

# 7. Icons snippets

## 7.1 File-based icons

Prefer file conventions when possible:

```txt
app/favicon.ico
app/icon.png
app/apple-icon.png
```

Use these when:

- the project just needs reliable, framework-native icon coverage
- there is no strong reason to generate icons in code

---

# 8. Pattern priorities by route type

## 8.1 Homepage

Prefer:

- root layout metadata
- root OG image fallback
- canonical `/`
- Organization/WebSite schema if applicable

## 8.2 Static marketing page

Prefer:

- static `metadata`
- route-level canonical
- inherited OG defaults unless page needs custom OG

## 8.3 Blog listing page

Prefer:

- static or lightly dynamic metadata depending on CMS/control needs
- canonical `/blog`
- dedicated OG if blog is a major landing area

## 8.4 Blog post from CMS/headless/API

Prefer:

- `generateMetadata`
- dynamic canonical
- dynamic OG image if content sharing matters
- inclusion in programmatic sitemap

## 8.5 Product detail page

Prefer:

- `generateMetadata`
- dynamic canonical
- product-specific OG strategy
- inclusion in programmatic sitemap
- product schema if implemented elsewhere in the project

## 8.6 Docs/article/help detail page

Prefer:

- `generateMetadata` if title/summary is content-driven
- static `metadata` only if docs content is fully stable and route-owned locally

## 8.7 Auth/account/cart/checkout

Prefer:

- no ranking intent
- careful robots/indexability decisions
- no accidental inclusion in sitemap

---

# 9. Anti-patterns to avoid

Do not prefer these patterns unless the project version or architecture truly requires them:

- generic static metadata on dynamic public entity pages
- missing `metadataBase` while using relative canonical/alternate/OG paths
- manual `<head>`-style fragmentation for App Router SEO
- one flat metadata object trying to represent all route types
- hardcoded production domain scattered across many files
- sitemap maintained manually while dynamic content changes frequently
- robots rules that do not reflect actual private/public route intent
- OG images handled entirely outside Next.js file conventions without good reason

---

# 10. Version notes

## 10.1 Next 15

When the project uses Next 15:

- prefer snippets compatible with App Router 15 docs
- verify examples against `/docs/15/...`
- be careful not to inject patterns that assume later APIs unless confirmed compatible

## 10.2 Latest/current Next

When the project uses the current major:

- prefer snippets from unversioned docs
- still verify compatibility with the project’s actual installed version

## 10.3 If the project version is unknown

If exact version cannot be confirmed from source:

- state that version could not be verified
- prefer conservative App Router patterns from official docs
- avoid newer edge-case APIs unless necessary

---

# 11. Documentation references to prioritize

Use these docs first when validating or adapting snippets:

## Latest/current

- `https://nextjs.org/docs/app/getting-started/metadata-and-og-images`

## Next 15

- `https://nextjs.org/docs/15/app/getting-started/metadata-and-og-images`

## Next 14

- `https://nextjs.org/docs/14/app/getting-started/metadata-and-og-images`

## Next 13

- `https://nextjs.org/docs/13/app/building-your-application/optimizing/metadata`

---

# 12. Final rule

Always choose snippets based on:

1. the project's actual Next.js version
2. the route type
3. whether metadata is static or data-driven
4. whether Next.js already provides a file convention for the job

Do not use a snippet just because it works.
Use the snippet that is most correct for:

- the installed Next.js version
- the route’s SEO behavior
- long-term maintainability
- official framework conventions
