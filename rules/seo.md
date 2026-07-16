# SEO Review Rules

## Purpose
Run this review when shipping content pages and before any public launch. The goal is to ensure the site can be discovered, indexed, and ranked — and that every page gives Google and users enough signal to understand what it is and why it matters.

SEO is structural, not magical. Ninety percent of the impact comes from five things: correct page titles, indexability, a sitemap, content structure, and Core Web Vitals. The rest is optimization after evidence.

Priority order: blockers must be fixed before launch. Critical findings must be fixed before merging SEO-critical pages. Warnings are optimization opportunities.

**Three invocation modes:**
- **New project**: run Step 0 (Project Scaffold) once before publishing any page
- **Content or feature page**: run Steps 1–4 before marking any SEO-visible page done
- **Pre-launch**: run Step 5 before any public launch or production deploy

---

## Severity Classification

**Blocker — must fix before launch:**
- `robots.txt` is blocking search engine crawlers — easy to introduce accidentally when a staging config reaches production
- No `sitemap.xml` — Google cannot reliably discover pages that aren't linked from the web
- Duplicate `<title>` tags across two or more pages — cannibalization actively suppresses rankings for both pages
- A page that should be indexed has a `noindex` directive (via `robots` metadata or HTTP header) — often introduced by copy-pasting a staging config
- `metadataBase` in the root layout is unset or pointing to localhost/staging in a production build — all canonical and OG URLs will be wrong

**Critical — must fix before merging an SEO-critical page:**
- Page has no `<title>` or the title is the default template value without a page-specific override
- Page has no meta description
- Page has no canonical URL — required on any page where the same content is reachable via more than one URL (with/without trailing slash, with query parameters, www vs. non-www)
- Page missing `og:image` — does not hurt ranking directly, but social sharing drives traffic and a missing image produces a blank card that no one clicks
- SEO-critical page (landing page, blog post, pricing) is dynamically rendered on every request with `cache: 'no-store'` — Google may see incomplete or slow content
- Dynamic route missing `generateMetadata` — metadata is being set client-side or not at all
- Core Web Vitals failing on a content page — Google uses all three as ranking signals (defer to performance.md for fixes, confirm pass here):
  - LCP above 2.5s (Largest Contentful Paint — how fast the main content loads)
  - CLS above 0.1 (Cumulative Layout Shift — how much content jumps after load)
  - INP above 200ms (Interaction to Next Paint — how fast the page responds to clicks; replaced FID in March 2024)

**Warning — flag before launch, fix before next deploy:**
- Title is outside 50–60 characters — truncation in search results reduces click-through rate
- Meta description is outside 150–160 characters — Google rewrites longer ones, often poorly
- No structured data on page types that support rich results (articles, FAQs, products)
- Internal links use non-descriptive anchor text ("click here", "read more") — anchor text is a relevance signal
- Page is not reachable via any internal link (orphan page) — Google may not crawl it even if it's in the sitemap
- `next/image` not used on above-the-fold images — affects LCP and Core Web Vitals

---

## Step 0: Project Scaffold (New Projects Only)

Run once when initializing a new project, before publishing any page. These files are set once and apply globally.

### 0a. robots.ts

In Next.js 14 App Router, create `app/robots.ts` to generate `robots.txt` dynamically. The critical rule: staging and preview deployments must not be indexed.

```ts
// app/robots.ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  const isProduction = process.env.VERCEL_ENV === 'production'

  return {
    rules: {
      userAgent: '*',
      allow: isProduction ? '/' : undefined,
      disallow: isProduction ? undefined : '/',
    },
    sitemap: `${process.env.NEXT_PUBLIC_SITE_URL}/sitemap.xml`,
  }
}
```

`VERCEL_ENV` is set automatically by Vercel to `"production"`, `"preview"`, or `"development"` — no domain string matching required. This correctly blocks all preview and development deployments regardless of the URL, including the auto-generated `*.vercel.app` preview URLs that Google can and will index if not blocked. Verify before launch: open `yourdomain.com/robots.txt` in an incognito browser and confirm it reads `Allow: /`.

### 0b. sitemap.ts

Create `app/sitemap.ts` to generate `sitemap.xml`. The function can be `async` — fetch dynamic slugs from the database and merge them with static routes. Content published after the initial build will only be discoverable by Google if it appears here.

Priority convention: `1.0` for home, `0.8–0.9` for primary landing pages, `0.7` for blog posts and secondary content, `0.5` for tag/category pages.

```ts
// app/sitemap.ts
import { MetadataRoute } from 'next'
import { createClient } from '@/lib/supabase/server'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = process.env.NEXT_PUBLIC_SITE_URL!

  // Static routes — add every known page
  const staticRoutes: MetadataRoute.Sitemap = [
    { url: baseUrl, lastModified: new Date(), changeFrequency: 'monthly', priority: 1 },
    { url: `${baseUrl}/about`, lastModified: new Date(), changeFrequency: 'monthly', priority: 0.8 },
    { url: `${baseUrl}/pricing`, lastModified: new Date(), changeFrequency: 'monthly', priority: 0.9 },
  ]

  // Dynamic routes — fetch published slugs from the database
  const supabase = createClient()
  const { data: posts } = await supabase
    .from('posts')
    .select('slug, updated_at')
    .eq('published', true)

  const postRoutes: MetadataRoute.Sitemap = (posts ?? []).map(post => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: post.updated_at,
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  }))

  return [...staticRoutes, ...postRoutes]
}
```

### 0c. Root Metadata Template

In `app/layout.tsx`, set the metadata foundation that all pages inherit from:

```ts
// app/layout.tsx
export const metadata: Metadata = {
  metadataBase: new URL(process.env.NEXT_PUBLIC_SITE_URL!),
  title: {
    template: '%s | Product Name',
    default: 'Product Name — One-line description of what it does',
  },
  description: 'Default description — 150–160 characters. Used on pages that do not set their own.',
  openGraph: {
    siteName: 'Product Name',
    type: 'website',
    images: [{ url: '/opengraph-image.png', width: 1200, height: 630 }],
  },
  twitter: {
    card: 'summary_large_image',
  },
}
```

`metadataBase` is required. Without it, Next.js cannot construct absolute URLs for canonical and OG tags — relative paths will be rendered instead, which breaks social sharing and canonical signals.

### 0d. Default OG Image

Create a static fallback OG image at `public/opengraph-image.png` at 1200×630px. This is used on any page that does not override the OG image. A missing OG image renders as a blank card on every social share. Every page must have one — the default covers pages that don't set their own.

For key pages (home, landing), create page-specific OG images using Next.js's built-in `opengraph-image.tsx` co-located with the page, or generate them dynamically with `ImageResponse`.

### 0e. Utility Page noindex Convention

Pages that users should not find via search must be explicitly blocked. Without this, Google indexes thin or private content and treats it as a quality signal for the whole site.

**Always add `robots: { index: false }` to:**
- Post-conversion pages: `/thank-you`, `/success`, `/payment-confirmed`
- Auth flow pages: `/auth/callback`, `/auth/verify`, `/reset-password`
- Onboarding steps: `/onboarding/[step]`
- Internal or admin pages: `/dashboard`, `/settings`, `/admin/*`
- Preview or draft pages: `/preview/[token]`

In the page's `metadata` export:

```ts
export const metadata: Metadata = {
  title: 'Thank You',
  robots: { index: false },
}
```

Set this at the layout level for any route group that should never be indexed (e.g., `(dashboard)/layout.tsx`) so it applies automatically to every page in the group rather than requiring it per-page.

### 0f. Scaffold Sign-off

Before writing any feature code:
- [ ] `app/robots.ts` exists, uses `VERCEL_ENV === 'production'`, and blocks all non-production environments
- [ ] `app/sitemap.ts` exists, is `async`, and includes both static routes and a dynamic database fetch pattern
- [ ] `metadataBase` is set in `app/layout.tsx` to the production URL
- [ ] Title template is configured: `'%s | Product Name'`
- [ ] Default OG image exists at `public/opengraph-image.png`
- [ ] `NEXT_PUBLIC_SITE_URL` is set correctly in production environment variables on Vercel
- [ ] Utility page route groups (dashboard, auth, onboarding) have `robots: { index: false }` at the layout level

---

## Step 1: Page Metadata Check (Per Content Page)

Run this for every new or significantly updated page before marking it done.

**Title:**
- [ ] The page has a unique title — not the same as any other page on the site
- [ ] Title is set via `export const metadata` (static pages) or `generateMetadata` (dynamic pages) — never client-side
- [ ] Title is 50–60 characters including the template suffix (e.g., "Consulting Services | Douglas Wang")
- [ ] Title describes the specific page — not the site name repeated or a generic label

**Description:**
- [ ] The page has a unique meta description — not the same as any other page
- [ ] Description is 150–160 characters
- [ ] Description is written for a human scanning search results: what the page is about and why it matters

**Canonical URL:**
- [ ] `alternates.canonical` is set to the definitive URL for this page
- [ ] The canonical URL is absolute (includes `https://` and the domain) — `metadataBase` handles this if set correctly
- [ ] If this page is reachable via multiple URLs (with/without trailing slash, with `?utm_source=...` parameters), the canonical points to the clean version

**Open Graph:**
- [ ] `og:title` — either set explicitly or inheriting the page title. Confirm it reads correctly
- [ ] `og:description` — either set explicitly or inheriting the meta description
- [ ] `og:image` — either a page-specific image or the site default. Must be 1200×630px minimum
- [ ] `og:type` — `website` for landing/informational pages, `article` for blog posts

**Twitter:**
- [ ] `twitter:card` is `summary_large_image` — the format that shows the full image preview
- [ ] `twitter:image` is set — inherited from OG image if not overridden explicitly

**Dynamic routes:**
- [ ] Pages at `/blog/[slug]`, `/products/[id]`, or similar use `generateMetadata` to pull title and description from the data source — not hardcoded
- [ ] `generateMetadata` returns a fallback for slugs that don't exist (404 pages should still have a title)

---

## Step 2: Content Structure Check (Per Content Page)

**Headings:**
- [ ] Exactly one `<h1>` per page that matches or is closely related to the page title
- [ ] Headings follow a logical hierarchy — `<h2>` for sections, `<h3>` for subsections. No skipped levels
- [ ] Headings describe the content that follows — not used purely for visual sizing (cross-reference: accessibility.md Step 3)

**URLs:**
- [ ] The page URL slug is descriptive and lowercase: `/consulting-services`, not `/page-3` or `/ConsultingServices`
- [ ] URL uses hyphens, not underscores: `/case-studies`, not `/case_studies`
- [ ] URL does not contain query parameters in the canonical path — filter/sort parameters are excluded from the canonical

**Internal linking:**
- [ ] The page is linked from at least one other page on the site — no orphan pages
- [ ] Anchor text is descriptive: "read our pricing guide" — not "click here" or "learn more"
- [ ] Key pages (home, primary landing pages) are linked from the navigation or footer — not only reachable via the sitemap

**Images:**
- [ ] All meaningful images have `alt` text that conveys meaning (cross-reference: accessibility.md Step 8 — enforced there, confirmed here)
- [ ] Above-the-fold images use `<Image priority>` from `next/image` — critical for LCP

---

## Step 3: Rendering and Indexability Check (Per Content Page)

This is the most common Next.js-specific SEO failure. Google indexes what the server renders. Content that only appears after a client-side data fetch may not be indexed.

**Rendering mode:**
- [ ] SEO-critical pages (landing pages, blog posts, pricing, about) are statically rendered (SSG) or use ISR with a reasonable `revalidate` interval — not `{ cache: 'no-store' }` on every request
- [ ] Confirm by running `npm run build` and checking the route table in the output. Routes marked `○` (static) or `◐` (ISR) are correctly rendered. Routes marked `λ` (dynamic/server) on SEO-critical pages are a concern

**Dynamic routes:**
- [ ] `/blog/[slug]` and similar content routes use `generateStaticParams` to pre-render known slugs at build time
- [ ] `export const dynamicParams = true` is set (or left as default) alongside `generateStaticParams` — this ensures slugs published after the build are rendered on-demand and cached, not served as 404s. Setting it to `false` means any new content requires a full redeploy before it becomes accessible
- [ ] If content updates frequently and full static generation is impractical, ISR with `revalidate: 3600` (hourly) is the correct fallback — not dynamic rendering

**Indexability:**
- [ ] The page does not have `robots: { index: false }` in its metadata unless intentionally hidden (e.g., thank-you pages, internal tools)
- [ ] The page is not behind authentication if it is intended to be indexed — Google cannot log in
- [ ] Next.js middleware does not redirect unauthenticated users away from pages that should be publicly indexed

**Core Web Vitals:**
- [ ] Run Lighthouse on the production URL (or a production-like preview): SEO score 90+, Performance score 70+ minimum
- [ ] LCP under 2.5s — if failing, defer to performance.md Step 5 for fixes
- [ ] CLS under 0.1 — images have explicit dimensions; no layout shifts from dynamic content loading
- [ ] INP under 200ms — heavy `onClick` handlers or large client-side state updates are the common cause; defer to performance.md if failing
- [ ] All three must show green in Lighthouse — a partial pass (two green, one yellow) does not qualify for Google's Core Web Vitals ranking boost

---

## Step 4: Structured Data Check (Per Content Page)

Structured data enables rich results in Google Search — expanded listings that show star ratings, article dates, FAQ dropdowns, and more. These increase click-through rates without changing rankings.

Only implement structured data types that match the actual content. Do not add schema speculatively.

**Which schema type applies to this page:**

| Page type | Schema type | What it enables |
|---|---|---|
| Personal brand / portfolio | `Person` | Name, role, social links in knowledge panel |
| Company / product site | `Organization` | Logo, contact info, social links in knowledge panel |
| Blog post / article | `Article` | Publication date, author, headline in results |
| FAQ or help page | `FAQPage` | Accordion of questions directly in search results |
| Landing page with a Q&A section | `FAQPage` | Same — high ROI for informational pages |
| Inner page with a hierarchy | `BreadcrumbList` | Breadcrumb trail shown under the URL in results |
| SaaS product | `SoftwareApplication` | Rating, category, operating system in results |

**Implementation in Next.js:**

Add structured data as a `<script>` tag in the page component — not in the `metadata` export, which does not support script injection:

```tsx
// Example: Article schema for a blog post
export default function BlogPost({ post }) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: { '@type': 'Person', name: 'Douglas Wang' },
    description: post.excerpt,
  }

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
      />
      {/* page content */}
    </>
  )
}
```

**Validation:**
- [ ] Test structured data with Google's Rich Results Test (search.google.com/test/rich-results) before marking done
- [ ] No warnings or errors in the test output
- [ ] The schema type matches the actual content — do not add `Article` schema to a landing page or `FAQPage` schema to a page with no FAQ section

---

## Step 5: Pre-launch Audit (Launch Gate)

Run this step once before any public launch and before pointing a custom domain at the deployment for the first time.

**Indexability verification:**
- [ ] Open `https://yourdomain.com/robots.txt` in an incognito browser — confirm it reads `Allow: /`, not `Disallow: /`
- [ ] Open `https://yourdomain.com/sitemap.xml` in an incognito browser — confirm it renders XML with the expected URLs, including any dynamic routes (blog posts, product pages)
- [ ] Confirm `metadataBase` in the root layout matches the production domain exactly — open any page, right-click → View Page Source, search for `og:image` and confirm the URL is absolute (starts with `https://yourdomain.com`), not a relative path
- [ ] Open the home page in an incognito browser and confirm it loads without errors or redirects

**Metadata sweep:**
- [ ] Open three pages: home, a key landing page, and one inner content page
- [ ] On each, right-click → View Page Source → search for `<title>` — confirm each has a unique, descriptive title
- [ ] On each, search for `og:image` — confirm an absolute URL is present (not a relative path)
- [ ] Paste each page URL into a social media link preview tool (e.g., opengraph.xyz) — confirm title, description, and image render correctly

**Lighthouse audit:**
- [ ] Run Lighthouse on the production URL for: home, primary landing page, and one content page
- [ ] SEO score: 90 minimum on each. Investigate and fix every flagged item
- [ ] Performance score: 70 minimum. Flag anything below — defer to performance.md for fixes but do not launch with a severely underperforming page

**Google Search Console (first launch only — requires live production domain):**
- [ ] Add property at search.google.com/search-console using the production domain
- [ ] Verify ownership via the HTML file method: download the verification file, place it in `public/`, deploy, then confirm verification in Search Console
- [ ] Submit sitemap: Search Console → Sitemaps → enter `sitemap.xml` → Submit
- [ ] Allow 24–48 hours, then check the Coverage report for indexing errors
- [ ] On subsequent launches (new features, redesigns): re-check Coverage for any newly introduced noindex or crawl errors

**Duplicate content:**
- [ ] Confirm `www.yourdomain.com` and `yourdomain.com` both resolve to the same canonical version (Vercel handles this with the `www` redirect setting — verify it is enabled)
- [ ] Confirm trailing slash behavior is consistent — `/about` and `/about/` should not both be indexable. Next.js defaults to no trailing slash; confirm `trailingSlash` in `next.config.js` matches canonical URLs

**Pre-launch sign-off:**
- [ ] robots.txt verified live in production
- [ ] sitemap.xml verified live in production
- [ ] metadataBase matches production domain
- [ ] Three pages verified for unique title, description, and OG image
- [ ] Lighthouse SEO 90+ on key pages
- [ ] Search Console property verified and sitemap submitted
- [ ] www / canonical redirect verified

Do not launch until all items above are checked. These failures are invisible until Google reports them — often weeks after launch.

---

## Step 6: SEO Sign-off

**Invocation type**: [new project scaffold / content page / pre-launch audit]
**Pages reviewed**: [list]

**Metadata** (title / description / canonical / OG / Twitter): [pass / issues — classify]
**Content structure** (H1 / headings / URLs / internal links): [pass / issues — classify]
**Rendering and indexability** (static/ISR, no noindex, Core Web Vitals): [pass / issues — classify]
**Structured data** (schema type / validation): [pass / not applicable — state why / issues — classify]
**Pre-launch audit** (robots / sitemap / metadataBase / Lighthouse / Search Console): [pass / not applicable — state why / issues — classify]

**Blockers**: [none / list — must fix before launch]
**Critical findings**: [none / list — must fix before merging]
**Warnings**: [none / list — flag for next deploy]

---

## Guiding Principles

- SEO is discovered, not assumed. Launch, submit the sitemap, and let Search Console tell you what Google sees. Optimize based on data, not predictions.
- The scaffold matters more than the optimization. A site with a correct `robots.txt`, a sitemap, unique titles, and fast pages will outperform a heavily optimized site that blocks crawlers or has duplicate titles.
- Rendering mode is the most missed Next.js SEO failure. If a landing page is dynamically rendered on every request, Google may see a slower or incomplete version. Default to static or ISR for anything Google should index.
- Do not add structured data speculatively. Add it for the page types you actually have. One valid, relevant schema block beats five schema types with warnings.
- Core Web Vitals are a ranking signal. A page that loads slowly or shifts layout on load is actively penalized. Performance and SEO are not separate concerns.
