---
name: market-gap-research
description: Use proactively whenever the user mentions ANY form of competitor research, complaint mining, product gap analysis, SaaS wedge hunting, or "what are people complaining about X". Trigger phrases include "find gaps in [product]", "what are people saying about [tool]", "research [product] for missing features", "where can I build a wedge against [competitor]", "make me an Instagram/LinkedIn post about gaps in [product]", "use Apify to research [product]", "what do users wish [product] did", "find product opportunities in [category]", "reverse-engineer [competitor] complaints", "help me find a SaaS idea around [category]", "review [product]", "is [product] worth it", "[product] alternatives". Be liberal — fire even when the user does not say "gap" or "wedge".
---

# Market Gap Research

Mine X (Twitter), Reddit, and review sites via Apify to find the gaps, complaints, and unmet feature requests around a target product. Synthesise findings into an Instagram carousel, LinkedIn post, or research report — each gap backed by a direct quote with attribution.

This skill encodes the workflow Parikshit uses for both client work (Accelerate AI) and personal content (`@parikshitpruthi`, `@parikshitbuilds`). Research-as-content fits the positioning, so the output is always two things at once: an internal artifact AND a publishable one.

---

## When to fire

Fire whenever the user mentions any flavour of competitor research, complaint mining, or product gap analysis — even if they don't use the words "gap" or "wedge". Be liberal. See the `description` frontmatter for the full trigger list.

**Do NOT fire** for:
- General market sizing or TAM analysis (this skill is about *qualitative* gaps from real user voice, not numbers)
- Pricing research alone (use the rag-web-browser directly)
- Anything where the user already knows the gap and just wants to build it

---

## Prerequisite: Apify MCP

This skill assumes the **Apify MCP server is connected**, which exposes:

- `Apify:search-actors` — find scrapers
- `Apify:fetch-actor-details` — see input schema and pricing
- `Apify:call-actor` — run a scraper
- `Apify:get-actor-output` — fetch results
- `Apify:apify--rag-web-browser` — Google + page-fetch in one call (use heavily)

If these tools are not available, **stop and tell the user**:

> "I need the Apify MCP server connected to run this skill. Add it via Claude Settings → Connectors → Apify, then re-run."

Do not try to substitute with WebFetch or WebSearch. The Reddit and X scrapers are the actual signal source.

---

## Workflow

### Step 0 — Confirm scope with the user (one quick check)

Ask once: **"Should I focus on B2B SaaS-style complaints, or do you also want consumer/Instagram-style chatter?"** This determines whether to skip Instagram (default for B2B SaaS) and whether to weight Reddit > X or vice versa.

If the user has already told you what they want (e.g. "make me an Instagram carousel about gaps in waitlister.me"), skip the question — they've answered it.

### Step 1 — Baseline understanding (always run this first)

Before scraping for complaints, ground the rest of the analysis. Run:

```
Apify:apify--rag-web-browser
  query: "<product> features pricing what is it"
  maxResults: 3
```

Skim the results to confirm what the product actually is, who runs it, what tier it's in, and what category. Without this, you'll mistake unrelated noise for signal in the next steps.

If the product is a person/creator, search for their main asset/site instead (e.g. "Devin DaymoreDev waitlister.me" rather than "Devin").

### Step 2 — Pick the right Apify actors

Use `Apify:search-actors` with **broad keywords** (just the platform name, not "X scraper"). Run at least two searches per platform to compare. Pick the actor with:

1. Lowest **price-per-result** (watch for tiered vs flat pricing).
2. Highest **monthly user count** and success rate.
3. An input schema that returns **post content + comments**, not just metadata.

For most runs, these defaults work. Only re-search if they're unavailable:

| Source | Default actor | Pricing | Why |
|---|---|---|---|
| Reddit | `clearpath/reddit-search-scraper` | $0.00099/result | Returns posts AND comments, supports explicit subreddits |
| X / Twitter | `simoit/x-twitter-search-tweets-scrapper` | $0.0003/result | Cheapest viable, supports search terms with operators |
| Web (reviews, comparisons, AppSumo, G2) | `apify/rag-web-browser` | per query | Already in MCP, returns markdown-formatted content |
| Instagram | `apify/instagram-search-scraper` | per result | Only use if user explicitly asks for IG data |

Full input schemas live in [`references/apify-actors-cheatsheet.md`](./references/apify-actors-cheatsheet.md). Read that file before your first `call-actor` of a run.

### Step 3 — Run scrapers in this order (don't parallelise)

Cost adds up and you may need to refine queries between runs. Order matters:

1. **Reddit first** — highest signal density for product complaints.
2. **X second** — triangulates and surfaces the "silent product" signal (see Step 5).
3. **Review sites last** — usually the goldmine, but you only know what to look for after social listening.

### Step 4 — Reddit gotcha: auto-discovery is dangerous

`autoDiscoverSubreddits: true` is dangerous for products with **generic names** (waitlist, stack, pulse, loop, notion-lite, etc.). Reddit will return posts about college admissions waitlists, Rolex waitlists, Instacart waitlists, Notion alternatives — none useful.

**Rule:** if the product's name overlaps with a common English word, set `autoDiscoverSubreddits: false` and pass an explicit `subreddits` array.

Default SaaS subreddit set:

```json
["SaaS", "indiehackers", "Entrepreneur", "startups", "SideProject",
 "EntrepreneurRideAlong", "microsaas", "nocode", "webdev", "marketing",
 "smallbusiness", "growmybusiness"]
```

Add by category:

| Category | Add these subreddits |
|---|---|
| E-commerce tools | `ecommerce`, `Shopify`, `WooCommerce`, `FulfillmentByAmazon` |
| Dev tools | `programming`, `Frontend`, `javascript`, `webdev`, `learnprogramming` |
| Real estate | `realtors`, `RealEstate`, `PropTech`, `realestateinvesting` |
| AI / LLM tools | `LocalLLaMA`, `MachineLearning`, `OpenAI`, `singularity` |
| Design tools | `UXDesign`, `web_design`, `graphic_design`, `Figma` |
| Marketing / growth | `marketing`, `digital_marketing`, `growmybusiness`, `PPC`, `SEO` |

For products with **distinctive names** (Perplexity, Linear, Cursor, Vercel), `autoDiscoverSubreddits: true` is fine and will surface more.

Recommended Reddit input:

```json
{
  "query": "<product> OR <product variant>",
  "maxResults": 60,
  "contentType": "both",
  "sort": "relevance",
  "subreddits": ["SaaS", "indiehackers", "..."],
  "autoDiscoverSubreddits": false
}
```

### Step 5 — X gotcha: recognise the "silent product" signal

For most B2B SaaS tools, ~100% of tweets containing the product name are users sharing **their own pages built with it** (signup links, waitlist links, product-launched-via-X tweets). Zero complaints, zero discussion.

**This is itself a finding.** Report it to the user explicitly:

> "X is silent about [product]. Of the N tweets I pulled, all are users sharing pages they built with it — no complaints, no comparisons. That's a signal: the product is used quietly. The actual chatter lives on Reddit and review sites."

Do **not** hide this. It's credibility-building.

To find the rare-but-high-signal X chatter, run a **second search** with operators:

```
"<product> alternative" OR "<product> vs" OR "better than <product>" OR "<product> is" OR "switching from <product>"
```

This rarely returns more than a handful of results, but those are gold — the moments competitors and unhappy users speak publicly.

### Step 6 — Review sites are the goldmine

Run:

```
Apify:apify--rag-web-browser
  query: "<product> review limitations cons missing features"
  maxResults: 5
```

This typically surfaces:

- **AppSumo** (verified-purchase reviews — single highest-signal source for indie SaaS, often with founder responses underneath)
- **G2 / Capterra** (more enterprise; lower signal for indie tools)
- **The product's own "honest comparison" or "alternatives" page** — founders who write these admit specific weaknesses. Most credible source for gaps because it comes from the operator themselves.
- **Comparison blog posts** ("X vs Y", "best alternatives to X")

If the product has fewer than ~10 reviews on AppSumo/G2, the rag-web-browser may not surface much. In that case, do a second query: `"<product>" site:reddit.com` to find any Reddit threads the scoped Reddit search missed.

### Step 7 — Skipping Instagram (default)

For B2B SaaS gap research, **default to skipping Instagram**. Instagram scrapers return hashtag info, post metadata, and profile data — but the comment-level discussion where complaints actually live isn't accessible. Burning credits on it produces noise.

Tell the user explicitly:

> "Skipping Instagram — its scrapers return hashtag/profile metadata, not comment-level complaints. The real chatter is on Reddit and review sites. Let me know if you specifically want IG hashtag/profile data."

Only run `apify/instagram-search-scraper` if the user pushes back or the product is consumer-facing (e.g. a fashion brand, a creator tool with strong IG presence).

### Step 8 — Synthesise gaps

Aim for **4–6 gaps**. Fewer than 4 = you didn't dig deep enough. More than 6 = you're diluting the report. Each gap MUST have:

1. A **direct quote**, with full attribution (source, author handle, date).
2. A **clear "wedge" callout** — what an opportunistic founder could build.
3. **Adjacent competitors** that already exist in the wedge (to size the opportunity).

Categorise each gap by type. Mix categories — don't ship 5 gaps all of the same type.

| Type | What it means |
|---|---|
| Use-case expansion | Product could serve adjacent jobs (waitlist tool → also do RSVPs, promos) |
| Customer journey | Gap before or after current scope (waitlist → no in-app referrals after launch) |
| UX / polish debt | Specific bugs and friction from review sites |
| Localization | Geography or language coverage |
| Architectural wedge | Headless vs hosted, API-first vs UI-first, OSS vs SaaS |

Also include a **"What's NOT a gap"** section at the end — counter-evidence the user expected but didn't find. This builds credibility and saves them a wasted bet.

### Step 9 — Choose output format

Ask the user, OR infer from context:

| Format | Spec |
|---|---|
| **Instagram carousel** | 9 slides, 1080×1350 portrait, single HTML file the user screenshots. Use the template in [`references/carousel-template.html`](./references/carousel-template.html). Brutalist editorial aesthetic — see Section "Carousel design" below. |
| **LinkedIn long-form post** | Markdown. Structure: hook → methodology credibility → 5 gaps with quotes → wedge callouts → CTA. ~1,200–1,800 words. |
| **Internal research report** | Markdown with full citations, raw evidence, "how to use this report" section. See [`examples/waitlister-gaps-report.md`](./examples/waitlister-gaps-report.md). |
| **Client deliverable** | Adapt to client branding/voice. Default: research report + a one-pager exec summary. |

**Always offer the markdown research report alongside the visual output.** It's reusable across IG, LinkedIn, YouTube, and pitch decks. The user explicitly wants both produced, every time.

---

## Carousel design (when output is Instagram)

The aesthetic that works for `@parikshitpruthi` and `@parikshitbuilds`:

- **Background:** warm off-white `#F2EEE5` (paper) with subtle grain texture
- **Typography:** Fraunces (serif, 900 weight, italic accent for emphasis) + Inter Tight (body) + JetBrains Mono (meta/numerals)
- **Accent palette (rotate per gap slide):** `#FF4D2E` (red-orange), `#0B5BFF` (electric blue), `#1F8F4A` (green), `#9B2BFF` (purple), `#D9A300` (mustard)
- **Frame:** 1080×1350 portrait. 48px outer margin, 2px black border, 56px inner padding.
- **Slide structure:** meta header (slide # + tag pill) → accent bar → display headline → body → quote box (left-bordered with accent, italic serif) → takeaway box (black background, white text, accent label) → footer (handle + page number)
- **Number treatment for gap slides:** oversized serif numeral (~280px) bleeding off the top-right of the slide in the gap's accent colour, low z-index so content sits over it.

Full template: [`references/carousel-template.html`](./references/carousel-template.html). Copy-paste, swap content, screenshot each slide, ship.

---

## Voice notes for captions / CTAs

Parikshit's tone: reflective, conversational, blends personal insight with industry commentary. Direct CTAs. Storytelling and humor.

- `@parikshitpruthi` — Hinglish, AI commentary, founder voice
- `@parikshitbuilds` — English, real estate AI builds
- LinkedIn — daily, 70% real estate / 30% founder voice

For carousel copy, lean **confident-but-not-bro**: "Find a real gap. Build the real fix." beats "10x your launch with these crazy hacks."

---

## Reference files (read these when...)

| File | Read when |
|---|---|
| [`references/apify-actors-cheatsheet.md`](./references/apify-actors-cheatsheet.md) | Before your first `call-actor` of any run, or when picking actors for a new platform |
| [`references/worked-example-waitlister.md`](./references/worked-example-waitlister.md) | When you want to see the full workflow play out on a real product (waitlister.me) — order of calls, queries used, decisions made |
| [`references/carousel-template.html`](./references/carousel-template.html) | When the output is an Instagram carousel — copy this, swap content slide-by-slide |
| [`examples/waitlister-gaps-report.md`](./examples/waitlister-gaps-report.md) | When you want to see what a finished research report looks like — use as template |
| [`examples/waitlister-gaps-carousel.html`](./examples/waitlister-gaps-carousel.html) | When you want to see what a finished 9-slide carousel looks like — copy and adapt |

---

## Common mistakes (and fixes)

| Mistake | Symptom | Fix |
|---|---|---|
| Running scrapers in parallel on first invocation | Burns $5+ before any signal validates the queries | Run sequentially. Reddit → X → web. Pause after each to skim. |
| Auto-discovery on a product with a generic name | Reddit returns 80 posts about college admissions / Rolex waitlists | Set `autoDiscoverSubreddits: false`. Pass explicit `subreddits` array. |
| Treating the "silent product" signal as a failed scrape | You report "no data found" and the user thinks the run failed | Report it as a finding: "X is silent about this product, here's what that means." |
| Skipping the baseline (Step 1) | You misread unrelated tweets as complaints | Always run rag-web-browser for the product overview first. |
| Fewer than 4 gaps | Report feels thin | Dig deeper into review sites. Run a second rag-web-browser query with different phrasing. |
| More than 6 gaps | Carousel can't hold the structure; LinkedIn post bloats | Cut to the strongest 5. Move the rest to "honourable mentions" in the report. |
| Quotes without attribution | Loses credibility on LinkedIn | Every quote needs source + author handle + date. No exceptions. |
| Running Instagram scraper for B2B SaaS | $5+ spent on hashtag metadata that surfaces no complaints | Default to skipping IG for B2B. Tell the user why. |

---

## Output checklist (before shipping)

- [ ] Baseline (Step 1) confirmed product is what you think it is
- [ ] At least one Reddit + one X + one web query run
- [ ] X "silent product" signal explicitly addressed (positive or negative finding)
- [ ] 4–6 gaps identified, each with quote + attribution + wedge + adjacent competitors
- [ ] Categories mixed (not all UX-debt, not all use-case expansion)
- [ ] "What's NOT a gap" section included
- [ ] Markdown research report produced
- [ ] Visual deliverable produced (if user asked) using template
- [ ] Footer/CTA matches the user's voice (Hinglish for `@parikshitpruthi`, English for `@parikshitbuilds`)
