# Next.js SEO Snippets

Use this file as the implementation reference for SEO-related fixes in Next.js projects.

## Rules

- Prefer official Next.js APIs and patterns
- Prefer minimal edits
- Prefer App Router native patterns when `app/` exists
- Prefer metadata derived from the same source as the page content
- For blog, article, product, category, docs, and slug pages, prefer `generateMetadata`
- For public SEO pages, prefer static generation or ISR when freshness allows
- Use request-time rendering only when the content truly requires per-request freshness
- If a Vercel skill is available, it may be used to evaluate SSR, SSG, ISR, caching, and deployment-aware rendering choices

---

## 1. Metadata and OG Images

> **Reference:** https://nextjs.org/docs/app/getting-started/metadata-and-og-images

**Use for:**

- `title`, `description`, `canonical`
- Open Graph metadata
- Twitter metadata
- Static metadata export

**When to use:**

- Static pages
- Stable pages whose SEO fields are known without route params or API fetches

```ts
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About',
  description: 'Learn more about our company',
  alternates: {
    canonical: '/about',
  },
  openGraph: {
    title: 'About',
    description: 'Learn more about our company',
    url: '/about',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'About',
    description: 'Learn more about our company',
  },
};
```

---

## 2. Dynamic Metadata with `generateMetadata`

> **Reference:** https://nextjs.org/docs/app/api-reference/functions/generate-metadata

**Use for:**

- Blog posts
- Articles
- Product pages
- Category pages
- Docs pages
- Slug routes
- Any route where metadata depends on params or fetched data

**When to prefer:**

- The page content comes from API, CMS, or database
- SEO fields depend on slug or route params
- The page is SEO-critical and content-driven

```ts
import type { Metadata } from 'next';

type PageProps = {
  params: Promise<{ slug: string }>;
};

type Post = {
  title: string;
  excerpt?: string;
  seoTitle?: string;
  seoDescription?: string;
  ogImage?: string;
  publishedAt?: string;
};

async function getPost(slug: string): Promise<Post | null> {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/posts/${slug}`, {
    next: { revalidate: 3600 },
  });

  if (!res.ok) return null;
  return res.json();
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);

  if (!post) {
    return {
      title: 'Post not found',
      description: 'The requested post could not be found.',
    };
  }

  const title = post.seoTitle || post.title;
  const description = post.seoDescription || post.excerpt || 'Read this article.';

  return {
    title,
    description,
    alternates: {
      canonical: `/blog/${slug}`,
    },
    openGraph: {
      title,
      description,
      url: `/blog/${slug}`,
      type: 'article',
      images: post.ogImage ? [{ url: post.ogImage }] : [],
      publishedTime: post.publishedAt,
    },
    twitter: {
      card: 'summary_large_image',
      title,
      description,
      images: post.ogImage ? [post.ogImage] : [],
    },
  };
}
```

---

## 3. Merging Parent Metadata

> **Reference:** https://nextjs.org/docs/app/api-reference/functions/generate-metadata

**Use for:**

- Nested layouts
- Extending shared metadata safely

```ts
import type { Metadata, ResolvingMetadata } from 'next';

type PageProps = {
  params: Promise<{ slug: string }>;
};

export async function generateMetadata(
  { params }: PageProps,
  parent: ResolvingMetadata
): Promise<Metadata> {
  const { slug } = await params;
  const previous = await parent;

  return {
    title: `Article: ${slug}`,
    openGraph: {
      images: ['/blog-default-og.jpg', ...(previous.openGraph?.images || [])],
    },
  };
}
```

---

## 4. Open Graph Image Generation

> **Reference:** https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image

**Use for:**

- Page-specific social images
- Generated share images
- Route-level OG image handling

```ts
import { ImageResponse } from 'next/og';

export const size = {
  width: 1200,
  height: 630,
};

export const contentType = 'image/png';

export default function Image() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 64,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        Blog
      </div>
    ),
    {
      ...size,
    }
  );
}
```

---

## 5. Robots

> **Reference:** https://nextjs.org/docs/app/api-reference/file-conventions/metadata/robots

**Use for:**

- `robots.txt` generation
- Crawl rules configuration
- Sitemap reference

```ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/admin/', '/api/internal/'],
    },
    sitemap: 'https://example.com/sitemap.xml',
  };
}
```

---

## 6. Sitemap

> **Reference:** https://nextjs.org/docs/app/api-reference/file-conventions/metadata/sitemap

**Use for:**

- Sitemap generation
- Dynamic public URLs
- Canonical page discovery

```ts
import type { MetadataRoute } from 'next';

type Post = {
  slug: string;
  updatedAt?: string;
};

async function getPosts(): Promise<Post[]> {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/posts`, {
    next: { revalidate: 3600 },
  });

  if (!res.ok) return [];
  return res.json();
}

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getPosts();

  const postUrls = posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt ? new Date(post.updatedAt) : new Date(),
  }));

  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
    },
    {
      url: 'https://example.com/blog',
      lastModified: new Date(),
    },
    ...postUrls,
  ];
}
```

---

## 7. JSON-LD

> **Reference:** https://nextjs.org/docs/app/guides/json-ld

**Use for:**

- `Organization`
- `WebSite`
- `BreadcrumbList`
- `Article`
- `Product`
- `FAQPage`

### Generic `JsonLd` component

```tsx
export function JsonLd({ data }: { data: Record<string, unknown> }) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data),
      }}
    />
  );
}
```

### Article JSON-LD example

```tsx
type ArticleJsonLdProps = {
  url: string;
  headline: string;
  description: string;
  image?: string;
  datePublished?: string;
  dateModified?: string;
};

export function ArticleJsonLd({
  url,
  headline,
  description,
  image,
  datePublished,
  dateModified,
}: ArticleJsonLdProps) {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    mainEntityOfPage: url,
    headline,
    description,
    image: image ? [image] : undefined,
    datePublished,
    dateModified,
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data),
      }}
    />
  );
}
```

---

## 8. `generateStaticParams`

> **Reference:** https://nextjs.org/docs/app/api-reference/functions/generate-static-params

**Use for:**

- Known dynamic routes
- Public pages that can be prerendered
- Blog, docs, category, product routes when slugs are available ahead of time

```ts
type Post = {
  slug: string;
};

async function getPosts(): Promise<Post[]> {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/posts`, {
    next: { revalidate: 3600 },
  });

  if (!res.ok) return [];
  return res.json();
}

export async function generateStaticParams() {
  const posts = await getPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

---

## 9. ISR and Revalidation

> **References:**
> - https://nextjs.org/docs/app/guides/incremental-static-regeneration
> - https://nextjs.org/docs/app/guides/how-revalidation-works

**Use for:**

- Blog pages
- Docs
- Category pages
- Content that changes over time but does not need per-request freshness

```tsx
type Post = {
  title: string;
  content: string;
};

async function getPost(slug: string): Promise<Post | null> {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/posts/${slug}`, {
    next: { revalidate: 3600 },
  });

  if (!res.ok) return null;
  return res.json();
}

export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  const post = await getPost(slug);

  if (!post) {
    return <div>Not found</div>;
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

---

## 10. Request-Time Fresh Data

> **Reference:** https://nextjs.org/docs/app/getting-started/caching

**Use for:**

- Highly dynamic pages
- Request-dependent data
- Pages that truly need `no-store` fetching

```ts
async function getLiveData() {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/live`, {
    cache: 'no-store',
  });

  if (!res.ok) {
    throw new Error('Failed to fetch live data');
  }

  return res.json();
}
```

---

## 11. Images

> **Reference:** https://nextjs.org/docs/app/getting-started/images

**Use for:**

- Content images
- Editorial images
- Meaningful `alt` text

```tsx
import Image from 'next/image';

export function HeroImage() {
  return (
    <Image
      src="/blog/hero.jpg"
      alt="A preview image for the article"
      width={1200}
      height={630}
      priority
    />
  );
}
```

---

## 12. Base Site URL Helper

**Use for:**

- Canonical generation
- Sitemap URLs
- OG URL consistency

```ts
export function getSiteUrl(): string {
  return process.env.NEXT_PUBLIC_SITE_URL || 'http://localhost:3000';
}
```

---

## 13. Content-Type Preferences

### Blog / Article pages

- Prefer `generateMetadata`
- Prefer metadata from API or CMS
- Prefer `Article` JSON-LD
- Prefer ISR unless request-time freshness is truly required

### Product pages

- Prefer `generateMetadata`
- Prefer metadata from API or catalog source
- Prefer product-specific schema when appropriate
- Prefer ISR unless pricing or availability must always be request-fresh

### Marketing pages

- Prefer static `metadata`
- Prefer static generation
- Only use dynamic metadata when route-specific content actually requires it

### Docs pages

- Prefer `generateMetadata` for slug routes
- Prefer static generation or ISR depending on update frequency
- Keep canonical strategy consistent across versions and sections

---

## 14. Anti-Patterns

**Avoid:**

- One generic title and description for all dynamic pages
- Canonical pointing every page to the homepage
- Metadata not derived from API or CMS for data-driven pages
- Sitemap missing important public routes
- `robots` accidentally blocking public pages
- Schema that does not match the actual page content
- Full request-time rendering for public SEO pages without a clear reason
- Hardcoded production URLs when the project already uses env-based configuration
