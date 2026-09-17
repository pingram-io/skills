---
name: seo-decline
description: >-
  Produce a phased SEO game plan for a declining URL or keyword from weekly
  GSC/analytics reports, including live SERP reconnaissance (learn from what
  Google ranks). Covers blog refresh, new marketing pages, docs (MDX),
  blog/marketing content, YouTube, internal linking, technical checks, and
  realistic off-site tactics. Use for ranking recovery or growth on Pingram-style
  B2B SaaS sites.
---

# SEO decline (full game plan)

Turn around a **declining page or keyword** from a **weekly SEO report** (visitors, clicks, CTR, impressions, position). Deliver **one integrated strategy**, not a disconnected tactic list. Assume **limited leverage** on external listings unless clearly high-ROI.

## Inputs to require or infer

- **Target URL(s)** and **primary query intent** (informational, commercial, navigational, transactional).
- **Report deltas** (week-over-week or vs. prior period).
- **Brand vs. non-brand**; site-wide vs. page-specific decline.
- **Existing assets**: blog, docs, feature pages, comparisons, pricing, etc.
- **Conversion goal** (signup, doc read, demo).

State assumptions explicitly if inputs are missing.

## Phase 0 — Diagnose

1. **Classify the pattern**: demand drop; relevance drift; authority/cannibalization; technical; CTR collapse (SERPs, snippets, AIO).
2. **SERP reconnaissance (query Google for what actually ranks)**  
   Before rewriting anything, see the **live SERP** for the target cluster—GSC alone does not show winning angles or SERP features.
   - **Queries**: run the **head term** plus **3–8 variants** (questions, “how to”, “best”, “vs”, and phrasings straight from GSC “top queries” for the URL).
   - **Context**: use **correct locale** (country + language), check **mobile** as well as desktop; reduce personalization (private/incognito/signed-out) so results are not only “for you”.
   - **Log per keyword** (at least top **10** organic): URL, title, **content type** (guide, list, tool, docs, video, forum, product), **angle** (beginner vs expert, comparison, opinion), **freshness** cues (year in title, “updated”, changelog style).
   - **SERP widgets**: note **People Also Ask**, **related searches**, **featured snippets**, **videos**, **AI Overviews** / AIO—treat PAA/related as an **outline of intents** Google expects satisfied.
   - **Learn, don’t copy**: extract **H2-level patterns** (what sections winners share); find **gaps** (weak answers, missing steps, outdated stats) your page can own **better**.
   - **Internal link patterns on winners**: optional `site:example.com keyword` on strong domains to see how they hub/spoke.
   - **Automation**: if scripting, use **documented APIs** (e.g. Custom Search JSON API) and quotas; avoid brittle scraping.

3. **Internal**: cannibalization; weak internal links to/from the URL.

4. **Per-page GSC baseline (Pingram repo):** Pull **queries** (and **countries**) for **one pathname** vs the prior **Sun–Sat** week—the same weekly windows as `npm run seo` / diagnostics.
   - Command: **`npm run seo:page-report -- /blog/your-slug`** (pathname argument; defaults to **`/blog/how-to-send-emails-with-supabase`** if omitted).

   - Output: Prints tables and writes **`tools/seo/reports/<pathname-with-slashes-as-dashes>-gsc-phase0.md`** (e.g. `blog-how-to-send-emails-with-supabase-gsc-phase0.md`).

   - Auth: Same as other SEO tooling—**`npm run seo:auth`** once, **`GSC_SITE_URL`** optional (defaults `sc-domain:pingram.io`). Implementation: `tools/seo/gsc-page-query-report.ts`.

**Deliverable:** one **root-cause hypothesis** with **confidence** and what would **falsify** it, informed by **at least one** logged SERP (head + variants).

## Phase 1 — On-site content

### Refresh existing blog

Substantive update: intent-aligned H1/H2s; freshness (stats, screenshots, dates); depth (FAQ/PAA, steps, limits); `Article` / `FAQPage` schema where appropriate; TOC and UX. Specify **sections to add/rewrite/remove** and **success signals** (long-tail impressions, head-term position).

### New marketing pages (e.g. `website/src/pages/features/**`)

New page only for a **distinct cluster** or **persona**; avoid overlap—else **consolidate** or **canonical**. Plan slug, title, H1, primary/secondary keywords, internal links in/out.

### New docs (e.g. `website/src/content/docs/**/*.mdx`)

Task-based MDX: code samples, troubleshooting, links to **pillar blog** and **feature** pages; nav placement and API versioning if needed.

### New blog / marketing (`marketing/**` or repo blog path)

Gap-filling cluster content with **descriptive anchors** to the money URL; differentiation (examples, benchmarks) over volume.

**Discover real paths** in the workspace if they differ.

## Phase 2 — YouTube

One video tied to the page: title for YouTube + Google video; chapters; description + canonical link; embed only if it helps UX without hurting LCP unduly.

## Phase 3 — Internal linking

Hub/spoke: pillar vs. support; **5–15** proposed links (from → to, anchor **intent**). Breadcrumbs/footer if they clarify hierarchy.

## Phase 4 — Technical

Indexability, canonicals, redirects, CWV (especially after embeds), hreflang if multi-locale is real, sitemap + GSC after publish.

## Phase 5 — Off-site (realistic)

Prioritize **defensible** listings, partner integrations with real relationships, **PR** only with a genuine hook. Label **low-probability** outreach.

## Phase 6 — Measurement

**4- and 8-week** review: GSC by query/page; analytics landing events. **Leading indicators** before rank moves. **Kill/pivot criteria** if no recovery after fix + refresh.

## Required output shape

1. **Executive summary** (5 bullets: problem, hypothesis, primary bet, risks, 30-day focus).
2. **Backlog** P0/P1/P2 with owner type and dependencies.
3. **SERP learnings** (compact): queries checked; **patterns** winners share; **gaps** to exploit; notable **SERP features** (AIO, PAA, snippet).
4. **Outlines** only (unless asked for drafts): updated blog, new page, new doc, new post.
5. **Internal link map**.
6. **Metrics table** (metric, tool, baseline, target, review date).
7. **Non-goals**.

**Tone:** specific and opinionated; fewer **high-quality** ships; never promise “#1”; tie work to **intent** and **measurable** signals.
