# Performance Review Rules

## Purpose
Run this review before any deploy, and before merging any change that adds or modifies data-fetching, rendering, AI features, or database queries. The goal is to catch performance problems before users feel them — not after complaints arrive.

Priority order: blockers must be resolved before deploy. Critical issues must be resolved before merge. Warnings must be flagged and triaged.

Invoke explicitly: before any deploy, and before merging any data-fetching, rendering, or AI change.

---

## Severity Classification

**Blocker — stop deploy, fix immediately:**
- Unbounded query with no pagination, limit, or filter on a table that will grow (returns all rows forever)
- N+1 query in a loop: fetching inside a `.map()`, `forEach`, or any iteration over a collection
- Synchronous blocking operation on the server that pauses the entire response (large computation, unstreamed AI response on a slow AI call)
- Page or route that can never render because a required data fetch has no timeout or error boundary

**Critical — must fix before merge:**
- Fetching data sequentially that could be fetched in parallel (`await` inside a loop, or two independent `await` calls that block each other)
- `SELECT *` on a large table when only specific columns are needed
- Missing database index on a column used in a `WHERE`, `ORDER BY`, or `JOIN` clause on a query that will run frequently
- Rendering a large list without virtualization or pagination on the client
- Using a Client Component for a component that has no interactivity and could be a Server Component
- AI call that blocks the UI with no streaming, causing a blank or frozen state while waiting for a response
- Image served without Next.js `<Image>` optimization on a content-heavy page
- Third-party script (analytics, chat widget) loaded synchronously in `<head>` without `defer` or `strategy="lazyOnload"`

**Warning — flag before done, fix before next deploy:**
- Client Component that pulls in a large library only used in one place (could be lazy-loaded)
- Supabase RLS policy with a subquery or function call that runs on every row evaluated (policy-level N+1)
- AI context window significantly larger than the request requires (increases latency and cost)
- Repeated identical AI or API calls with no caching layer
- Static content regenerated on every request when `revalidate` or `cache` could be used
- Font loaded from an external CDN instead of via `next/font`
- No `loading.tsx` or `<Suspense>` boundary on a slow data-fetching route segment

---

## Step 0: Pre-flight Check

Before reviewing, establish scope:

1. **What changed?** Run `git diff main...HEAD --name-only` and identify files touching: data fetching, Server/Client components, API routes, Server Actions, Supabase queries, AI calls, or images/fonts/scripts.
2. **Is this a deploy review or a change review?** Deploy reviews cover the full app surface. Change reviews focus on the delta.
3. **Is there a known slow path?** If the user has reported slowness or a specific flow is known to be heavy, name it — that path gets extra scrutiny.

---

## Step 1: Rendering Architecture Review

For any change that adds or modifies components or page structure:

**Server vs Client Components:**
- [ ] Every component that has no browser-only requirement (event handlers, `useState`, `useEffect`, browser APIs) is a Server Component — not marked `"use client"`
- [ ] `"use client"` is pushed as far down the component tree as possible — it should wrap the interactive leaf, not the entire page
- [ ] No data fetching happens inside a Client Component when it could happen in a Server Component above it

**Streaming and Suspense:**
- [ ] Slow data-fetching operations (AI calls, heavy DB queries) are wrapped in `<Suspense>` so the rest of the page renders immediately
- [ ] `loading.tsx` exists for any route segment that has a meaningful wait time
- [ ] Users are never shown a blank screen while waiting — a skeleton, spinner, or partial content renders immediately

**Code splitting and lazy loading:**
- [ ] Large Client Components that are not needed on initial load are lazy-loaded with `next/dynamic`
- [ ] Heavy third-party libraries imported in Client Components are checked for bundle size impact — use `bundlephobia.com` or `@next/bundle-analyzer` if in doubt

---

## Step 2: Data Fetching Review

For any change that adds or modifies how data is fetched:

**Parallelization:**
- [ ] Independent data fetches run in parallel with `Promise.all()` — not sequentially with `await` one after the other
- [ ] No `await` inside a loop — batch fetch IDs first, then fetch all at once
- [ ] Server Component data fetching is initiated as high up the tree as possible to avoid request waterfalls between parent and child

**Fetch scope:**
- [ ] Queries select only the columns needed — not `SELECT *` when a subset is used
- [ ] List queries have a `limit` or pagination — no query returns unbounded rows
- [ ] Filters are applied at the database level, not in JavaScript after fetching all rows

**Caching:**
- [ ] Fetch calls in Server Components use Next.js cache options appropriately: `{ cache: 'force-cache' }` for static data, `{ next: { revalidate: N } }` for time-sensitive data, `{ cache: 'no-store' }` only when the data must be fresh on every request
- [ ] Repeated reads of the same data within a single request use React's automatic fetch deduplication (same URL + same options = one network call)
- [ ] Heavily repeated identical API calls (e.g., fetching the same user profile on every page) have a caching layer

---

## Step 3: Database Performance Review

For any change that adds or modifies Supabase queries:

**Indexes:**
- [ ] Every column used in a `.eq()`, `.filter()`, `.order()`, or join condition on a frequently-run query has an index
- [ ] New tables with foreign keys have indexes on the foreign key columns
- [ ] If unsure whether an index exists: run `SELECT indexname, indexdef FROM pg_indexes WHERE tablename = '<table>';` to check

**Query efficiency:**
- [ ] No N+1 pattern: if the code fetches a list and then fetches related data for each item individually, restructure as a single query with a join or a batch fetch
- [ ] Large result sets use `.range()` for pagination — not `.limit()` without an offset strategy
- [ ] Aggregations (counts, sums) are computed in the database, not by fetching all rows and counting in JavaScript

**RLS policy performance:**
- [ ] RLS policies do not contain subqueries that run per-row (e.g., `EXISTS (SELECT ...)` inside a policy on a large table) unless absolutely necessary
- [ ] If a policy subquery is unavoidable: confirm the subquery columns are indexed
- [ ] New policies are tested against realistic data volumes — a policy that works on 10 rows may be unacceptably slow on 10,000

---

## Step 4: AI/LLM Performance Review

Run this step for any change that adds, modifies, or calls the Claude API:

**Response streaming:**
- [ ] AI responses are streamed to the client — the user sees tokens as they arrive, not a blank UI until the full response is complete
- [ ] The stream is handled with proper error boundaries: a failed stream shows an error state, not a frozen spinner
- [ ] If streaming is intentionally not used: document why and confirm the typical response time is acceptable (under 2 seconds)

**Context efficiency:**
- [ ] The prompt sends only the data the model needs for this specific request — not the full user record, full conversation history, or all available context "just in case"
- [ ] Long conversation histories are truncated or summarized before being included in the prompt — unbounded history growth will eventually hit context limits and increase latency
- [ ] System prompts are not dynamically rebuilt on every request when a static or lightly parameterized version would work

**Caching and redundancy:**
- [ ] Identical or near-identical AI requests (same user, same prompt, within a short window) are not sent twice — a simple cache or deduplication check prevents redundant API calls
- [ ] If the same AI-generated output is shown to many users (e.g., a product description, a summary), it is generated once and stored — not regenerated per request
- [ ] Repeated system prompts or large context blocks use Anthropic prompt caching: add `cache_control: { type: "ephemeral" }` to the system prompt message when it exceeds 1,024 tokens and is reused across requests — this reduces cached token cost by ~90% and latency by ~50%

**Timeouts and fallbacks:**
- [ ] All AI API calls have a timeout — a hung request does not hang the user's session indefinitely
- [ ] A fallback or error state is shown if the AI call times out or fails — the UI does not wait forever

---

## Step 5: Frontend Performance Review

For any change that modifies pages, layouts, or significant UI components:

**Core Web Vitals:**
- [ ] **LCP (Largest Contentful Paint):** The largest visible element (hero image, heading, or content block) loads within 2.5 seconds. If it is an image: it uses `<Image priority>`. If it is text: the font is preloaded or uses `next/font`.
- [ ] **CLS (Cumulative Layout Shift):** No elements jump after initial render. Images have explicit `width` and `height` (or `fill` with a sized container). Dynamic content loads into reserved space, not pushing existing content.
- [ ] **INP (Interaction to Next Paint):** Interactive elements respond within 200ms. Long-running JavaScript on the main thread (heavy computation, large state updates) is deferred or moved off the main thread.

**Images:**
- [ ] All images use Next.js `<Image>` — not `<img>` tags
- [ ] Hero or above-the-fold images use `priority` prop
- [ ] Images served from external domains are listed in `next.config.js` `images.domains` and are not bypassing optimization

**Fonts:**
- [ ] Fonts are loaded via `next/font` — not via a `<link>` to Google Fonts or another CDN
- [ ] `font-display: swap` or `optional` is set to prevent invisible text during font load

**Third-party scripts:**
- [ ] Analytics, chat widgets, and marketing scripts use `next/script` with `strategy="lazyOnload"` or `strategy="afterInteractive"` — not placed in `<head>` without a strategy
- [ ] The total weight of third-party scripts is known — flag if any single script exceeds 100KB gzipped

---

## Step 6: Deploy Performance Checklist

Run this step only on deploy reviews.

**Build output:**
- [ ] Run `npm run build` and review the route size table in the output — flag any route whose First Load JS exceeds 250KB
- [ ] No route is accidentally importing a large server-side library into the client bundle (check for unexpected size spikes)

**Vercel configuration:**
- [ ] Routes that are purely static or can be statically generated are not using dynamic rendering unnecessarily
- [ ] API routes and Server Actions that have low latency requirements are on the Edge Runtime — routes that need Node.js APIs are on the Node.js runtime (not vice versa)
- [ ] Static assets (images, fonts, public files) are served from Vercel's CDN, not fetched from origin on every request

**Middleware matcher:**
- [ ] `middleware.ts` has an explicit `matcher` config — by default it runs on every request including static files, fonts, and images, adding unnecessary overhead to each
- [ ] The matcher excludes `_next/static`, `_next/image`, and `favicon.ico` at minimum
- [ ] The matcher covers all routes that require auth enforcement — verify no protected route is accidentally excluded

**Supabase connection:**
- [ ] The app uses Supabase connection pooling where appropriate — serverless environments (Vercel) should use the pooler connection string, not the direct connection string, to avoid exhausting connections under load

---

## Step 7: Performance Sign-off

Before marking done or proceeding with deploy, output this summary:

**Review type**: [deploy / pre-merge / data-fetching change / AI change / rendering change]
**Scope**: [list files and surfaces reviewed]

**Rendering architecture** (Server/Client split, Suspense): [pass / issues — classify]
**Data fetching** (parallelization, caching, scope): [pass / issues — classify]
**Database** (indexes, N+1s, pagination, RLS policies): [pass / issues — classify]
**AI/LLM performance** (streaming, context size, caching, timeouts): [pass / not applicable — explain / issues — classify]
**Frontend** (Core Web Vitals, images, fonts, scripts): [pass / issues — classify]
**Deploy checklist** (build output, Vercel config, DB connections): [pass / not applicable — explain / issues — list]

**Blockers**: [none / list — must resolve before deploy]
**Critical findings**: [none / list — must resolve before merge]
**Warnings**: [none / list — flag for triage]

---

## Guiding Principles

- Performance is not polish. A slow product loses users before they see the value.
- Measure before optimizing. Do not spend time on a fast path. Find the slow one first.
- The most common source of slowness in this stack: sequential data fetches, missing indexes, and unbounded AI context. Check these first.
- Streaming is not optional for AI features. A frozen UI while waiting for a model response is a broken product.
