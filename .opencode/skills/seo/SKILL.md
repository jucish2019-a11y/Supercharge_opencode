---
name: seo
description: Implement SEO for Next.js applications — metadata, Open Graph, structured data, sitemaps, robots.txt, performance, and crawlability
---

## What I do

I implement search engine optimization for Next.js web applications:

- **Metadata** — Title, description, canonical URLs, language alternates
- **Open Graph** — Social sharing previews (Twitter, LinkedIn, Facebook)
- **Structured data** — JSON-LD schemas for rich results
- **Sitemaps** — Dynamic sitemap generation and submission
- **robots.txt** — Crawl directives, sitemap location
- **Technical SEO** — Performance, Core Web Vitals, renderability, indexing
- **Analytics** — Google Analytics, Search Console integration

## When to use me

Use this skill when:
- Building a public-facing Next.js application that needs search traffic
- Setting up metadata and Open Graph for social sharing
- Adding structured data (JSON-LD) for rich search results
- Generating dynamic sitemaps
- Optimizing Core Web Vitals for SEO impact
- Debugging indexing or crawlability issues

## How I work

1. **Audit existing SEO** — Check metadata, structured data, sitemap, robots.txt, Web Vitals.
2. **Implement metadata layer** — Dynamic metadata per page using Next.js Metadata API.
3. **Add structured data** — JSON-LD for key content types.
4. **Generate sitemaps** — Dynamic based on database content.
5. **Configure robots.txt** — Allow crawling, block admin, reference sitemap.
6. **Optimize performance** — Core Web Vitals directly impact ranking.
7. **Set up analytics** — Google Analytics + Search Console.

## Metadata in Next.js App Router

### Static metadata

```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'ProjectTracker — Manage Projects Efficiently',
    template: '%s | ProjectTracker',
  },
  description: 'Manage projects, track tasks, and collaborate with your team. Simple, powerful project management.',
  metadataBase: new URL('https://www.projecttracker.com'),
  keywords: ['project management', 'task tracker', 'team collaboration'],
  authors: [{ name: 'ProjectTracker' }],
  creator: 'ProjectTracker',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://www.projecttracker.com',
    siteName: 'ProjectTracker',
    title: 'ProjectTracker — Manage Projects Efficiently',
    description: 'Manage projects, track tasks, and collaborate with your team.',
    images: [{ url: '/og-image.png', width: 1200, height: 630, alt: 'ProjectTracker' }],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'ProjectTracker — Manage Projects Efficiently',
    description: 'Manage projects, track tasks, and collaborate with your team.',
    images: ['/og-image.png'],
  },
  robots: {
    index: true,
    follow: true,
    googleBot: { index: true, follow: true, 'max-video-preview': -1, 'max-image-preview': 'large', 'max-snippet': -1 },
  },
  alternates: {
    canonical: 'https://www.projecttracker.com',
  },
};
```

### Dynamic metadata

```tsx
// app/projects/[id]/page.tsx
import type { Metadata } from 'next';

export async function generateMetadata({ params }): Promise<Metadata> {
  const project = await db.project.findUnique({ where: { id: params.id } });
  if (!project) return { title: 'Project Not Found' };

  return {
    title: project.name,
    description: project.description || `View the ${project.name} project on ProjectTracker.`,
    openGraph: {
      title: project.name,
      description: project.description,
      images: project.coverImage ? [{ url: project.coverImage }] : undefined,
    },
    alternates: {
      canonical: `https://www.projecttracker.com/projects/${params.id}`,
    },
  };
}
```

### Metadata rules

- Every page MUST have a unique title and description
- Title: 50-60 characters (Google truncates after ~60)
- Description: 150-160 characters (Google truncates after ~160)
- Canonical URL on every page to prevent duplicate content
- No duplicate titles across pages
- Title template ensures consistent branding: `%s | BrandName`

## Open Graph and social sharing

### Image requirements

```
Size:       1200×630px (Facebook, LinkedIn, Twitter large card)
Format:     PNG or JPG
File size:  < 1MB (faster sharing previews)
Text:       Minimal text on image (title only, large font)
Brand:      Include logo or brand mark
Background: Solid or gradient, not screenshot

Dynamic OG images:
  Use @vercel/og or satori to generate dynamic images
  Route: /api/og?title=Project+Name
  Returns: SVG-to-PNG image with the title rendered
```

### Dynamic OG image generation

```tsx
// app/api/og/route.tsx
import { ImageResponse } from '@vercel/og';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const title = searchParams.get('title') || 'ProjectTracker';

  return new ImageResponse(
    (
      <div style={{
        width: '100%', height: '100%',
        display: 'flex', alignItems: 'center', justifyContent: 'center',
        backgroundColor: '#1a1a2e', color: 'white', padding: '80px',
      }}>
        <div style={{ display: 'flex', flexDirection: 'column', gap: '24px' }}>
          <div style={{ fontSize: 72, fontWeight: 700 }}>{title}</div>
          <div style={{ fontSize: 28, color: '#a3a3a3' }}>projecttracker.com</div>
        </div>
      </div>
    ),
    { width: 1200, height: 630 }
  );
}
```

## Structured data (JSON-LD)

### Organization schema

```tsx
// In layout.tsx
const organizationSchema = {
  '@context': 'https://schema.org',
  '@type': 'Organization',
  name: 'ProjectTracker',
  url: 'https://www.projecttracker.com',
  logo: 'https://www.projecttracker.com/logo.png',
  sameAs: [
    'https://twitter.com/projecttracker',
    'https://github.com/projecttracker',
  ],
};

<script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(organizationSchema) }} />
```

### Software application schema

```tsx
const softwareSchema = {
  '@context': 'https://schema.org',
  '@type': 'SoftwareApplication',
  name: 'ProjectTracker',
  applicationCategory: 'BusinessApplication',
  operatingSystem: 'Web',
  offers: {
    '@type': 'Offer',
    price: '0',
    priceCurrency: 'USD',
  },
  aggregateRating: {
    '@type': 'AggregateRating',
    ratingValue: '4.8',
    ratingCount: '1247',
  },
};
```

### FAQ schema

```tsx
const faqSchema = {
  '@context': 'https://schema.org',
  '@type': 'FAQPage',
  mainEntity: [
    {
      '@type': 'Question',
      name: 'How do I create a project?',
      acceptedAnswer: {
        '@type': 'Answer',
        text: 'Click the "Create project" button on your dashboard...',
      },
    },
  ],
};
```

### Breadcrumb schema

```tsx
const breadcrumbSchema = {
  '@context': 'https://schema.org',
  '@type': 'BreadcrumbList',
  itemListElement: [
    { '@type': 'ListItem', position: 1, name: 'Home', item: 'https://www.projecttracker.com' },
    { '@type': 'ListItem', position: 2, name: 'Projects', item: 'https://www.projecttracker.com/projects' },
    { '@type': 'ListItem', position: 3, name: 'Project Alpha' },
  ],
};
```

## Sitemaps

### Dynamic sitemap (app router)

```tsx
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const projects = await db.project.findMany({
    where: { isPublic: true },
    select: { id: true, updatedAt: true },
  });

  const projectUrls = projects.map((project) => ({
    url: `https://www.projecttracker.com/projects/${project.id}`,
    lastModified: project.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  }));

  return [
    { url: 'https://www.projecttracker.com', lastModified: new Date(), changeFrequency: 'daily', priority: 1.0 },
    { url: 'https://www.projecttracker.com/pricing', lastModified: new Date(), changeFrequency: 'monthly', priority: 0.8 },
    { url: 'https://www.projecttracker.com/projects', lastModified: new Date(), changeFrequency: 'daily', priority: 0.9 },
    ...projectUrls,
  ];
}
```

### Sitemap rules

- Include all public, indexable pages
- Exclude: auth pages, admin, API routes, user-specific pages
- Set `lastModified` to actual content update time
- `priority`: homepage 1.0, key pages 0.8-0.9, content 0.6-0.7
- Keep under 50,000 URLs per sitemap (split if more)
- Submit to Google Search Console after generation

## robots.txt

```tsx
// app/robots.ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      { userAgent: '*', allow: '/', disallow: ['/api/', '/admin/', '/account/'] },
    ],
    sitemap: 'https://www.projecttracker.com/sitemap.xml',
  };
}
```

### robots.txt rules

- Allow all public content
- Disallow: `/api/`, `/admin/`, `/account/`, auth pages
- Reference sitemap location
- Don't disallow CSS/JS files (Google needs them for rendering)

## Technical SEO

### Core Web Vitals impact on ranking

Google uses Core Web Vitals as a ranking signal. Optimize these:

```
LCP (Largest Contentful Paint) — ≤2.5s
  - Optimize: next/image, server components, font loading, CDN
  - Measure: Lighthouse, Web Vitals in field

INP (Interaction to Next Paint) — ≤200ms
  - Optimize: reduce JavaScript, code splitting, web workers
  - Measure: Chrome User Experience Report

CLS (Cumulative Layout Shift) — ≤0.1
  - Optimize: image dimensions, font display: swap, ad slots
  - Measure: Lighthouse, Layout Shift in field
```

### Rendering and crawlability

```
Rules:
  - Use Server Components for content pages (SSR/SSG)
  - Don't render content behind client-side JavaScript (Googlebot executes JS but it's slower)
  - Use proper heading hierarchy (h1 → h2 → h3, no skipping)
  - Use semantic HTML (article, section, nav, main)
  - Ensure all content is in the HTML response (not API-loaded after render)
  - Internal linking: link between related pages (Google crawls links)
  - Clean URLs: /projects/project-name not /projects?id=123
```

### Page performance checklist

```
[ ] Server-rendered content (not client-only)
[ ] next/image for all images (proper sizing, lazy loading)
[ ] next/font for all fonts (no FOUT, no CLS)
[ ] No layout shift from lazy-loaded content
[ ] Proper heading hierarchy (h1 → h2 → h3)
[ ] Semantic HTML throughout
[ ] Meta robots not blocking indexing on public pages
[ ] Canonical URL on every page
[ ] No broken internal links
[ ] Mobile-responsive (mobile-first indexing)
```

## Analytics integration

### Google Analytics 4

```tsx
// lib/gtag.ts
export const GA_ID = process.env.NEXT_PUBLIC_GA_ID;

export function pageview(url: string) {
  window.gtag('config', GA_ID, { page_path: url });
}

export function event(action: string, params: Record<string, unknown>) {
  window.gtag('event', action, params);
}

// app/layout.tsx
import Script from 'next/script';

function Analytics() {
  return (
    <>
      <Script src={`https://www.googletagmanager.com/gtag/js?id=${GA_ID}`} strategy="afterInteractive" />
      <Script id="gtag" strategy="afterInteractive">
        {`window.dataLayer = window.dataLayer || [];
          function gtag() { dataLayer.push(arguments); }
          gtag('js', new Date());
          gtag('config', '${GA_ID}');`}
      </Script>
    </>
  );
}
```

### Google Search Console

```
1. Verify domain ownership (DNS TXT record or HTML tag)
2. Submit sitemap: https://www.projecttracker.com/sitemap.xml
3. Monitor: indexing status, search queries, Core Web Vitals
4. Review: coverage issues, mobile usability, structured data errors
5. Request indexing for new pages (URL inspection tool)
```

## SEO checklist

- [ ] Unique title and description on every page
- [ ] Title 50-60 chars, description 150-160 chars
- [ ] Canonical URL on every page
- [ ] Open Graph metadata with 1200×630 image
- [ ] Twitter card metadata
- [ ] JSON-LD structured data on key pages
- [ ] Dynamic sitemap for content pages
- [ ] robots.txt allows public content, blocks private
- [ ] Server-rendered content (not client-only)
- [ ] proper heading hierarchy (h1 → h2 → h3)
- [ ] Semantic HTML (article, section, nav, main)
- [ ] Internal linking between related pages
- [ ] Clean URLs (no query parameters for content)
- [ ] Core Web Vitals meet "good" thresholds
- [ ] Google Analytics and Search Console connected
- [ ] Mobile-responsive (Google uses mobile-first indexing)

## Anti-patterns I avoid

- Client-only rendering for content pages — Googlebot struggles with JS-heavy pages
- Missing title/description on any page — search engines will guess
- Duplicate titles across pages — Google can't distinguish them
- Blocking CSS/JS in robots.txt — Google needs them for rendering
- Using h1 for styling instead of heading hierarchy — use CSS for size
- Image without width/height — causes CLS (layout shift)
- No canonical URL — duplicate content penalty
- Auto-generated doorway pages — Google penalizes thin content
- Ignoring Core Web Vitals — they're a ranking signal
- No sitemap — Google discovers pages slower without one