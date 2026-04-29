# Worked example: gap research on waitlister.me

A walk-through of one full run, so future Claude can see the order of operations, the queries used, and the decisions made. The corresponding outputs are in [`../examples/waitlister-gaps-report.md`](../examples/waitlister-gaps-report.md) and [`../examples/waitlister-gaps-carousel.html`](../examples/waitlister-gaps-carousel.html).

## The user's prompt

> "Can you use the apify connector to find all the gaps that are there in waitlister.me? Look for all the gaps that you find on X, Reddit, Instagram, all these social media networks, and create a report which I can show on my Instagram on the gaps that are there in waitlister.me."

Inferred output: Instagram carousel + research report. Inferred scope: B2B SaaS (waitlister.me is a no-code waitlist tool). Inferred skip: Instagram (per default rule).

---

## Step 1 — Baseline

```
Apify:apify--rag-web-browser
  query: "waitlister.me features pricing what is it"
  maxResults: 3
```

Result: confirmed waitlister.me is a no-code waitlist tool. $15/mo entry, free tier, AI page builder, 30+ integrations. Founder is Devin (`@DaymoreDev`). Indie/bootstrapped.

This grounded the rest of the run. Every later piece of evidence got interpreted through "this is an indie no-code waitlist builder," not "this is a generic SaaS."

## Step 2 — Actor selection

`Apify:search-actors` with `"reddit"`, `"twitter"`, `"x"`, `"instagram"`. Compared 2–3 candidates each. Picked:

- Reddit → `clearpath/reddit-search-scraper` ($0.00099/result, returns comments)
- X → `simoit/x-twitter-search-tweets-scrapper` ($0.0003/result)
- Instagram → skipped (default rule for B2B SaaS)
- Web → `apify/rag-web-browser` (already in MCP)

## Step 3 — Reddit, first attempt (the trap)

```json
{
  "query": "waitlister.me OR waitlister",
  "maxResults": 80,
  "contentType": "both",
  "autoDiscoverSubreddits": true
}
```

Result: 80 posts. Most about **college admissions waitlists, Rolex waitlists, Instacart driver waitlists**. Maybe 5 SaaS-relevant. **Useless.**

Diagnosis: "waitlist" is a generic English word. Auto-discovery surfaced the wrong communities.

## Step 4 — Reddit, scoped retry

```json
{
  "query": "waitlister.me OR waitlister",
  "maxResults": 60,
  "contentType": "both",
  "sort": "relevance",
  "subreddits": [
    "SaaS", "indiehackers", "Entrepreneur", "startups", "SideProject",
    "EntrepreneurRideAlong", "microsaas", "nocode", "webdev", "marketing",
    "smallbusiness", "growmybusiness"
  ],
  "autoDiscoverSubreddits": false
}
```

Result: 60 SaaS-specific posts. Found a key thread from the founder of `pleasehold.dev` positioning his product as the "headless API-first" alternative to waitlister.me — Gap 5.

## Step 5 — X, name search

```json
{
  "searchTerms": "waitlister.me",
  "sort": "top",
  "maxItems": 30
}
```

Result: 20 tweets. **All** users sharing their own waitlist signup pages. Zero complaints. Zero comparisons.

This is the **silent product** signal. Reported to the user as a finding, not a failure:

> "X is silent about waitlister.me. All 20 tweets are users sharing their own pages — no complaints, no comparisons. This tells us the product is used quietly by builders, and the actual chatter lives on Reddit and review sites."

## Step 6 — X, complaint operators

```json
{
  "searchTerms": "\"waitlister vs\" OR \"better than waitlister\" OR \"waitlister alternative\"",
  "sort": "top",
  "maxItems": 20
}
```

Result: 1 organic comparison question — proof that waitlister.me has become the default reference point in its category. Used as a credibility bullet, not a gap.

## Step 7 — Review sites (the goldmine)

```
Apify:apify--rag-web-browser
  query: "waitlister.me review limitations cons missing features"
  maxResults: 5
```

Surfaced:

- **AppSumo listing** — 4.5★, 13 verified-purchase reviews. Multiple explicit feature requests with the founder responding underneath.
- **The founder's own "honest comparison" page** on waitlister.me — admitting weaknesses vs alternatives.

This was the highest-signal source by far. 4 of the 5 final gaps came from here.

## Step 8 — Synthesise

Five gaps, each with quote + attribution + wedge + adjacent competitors:

| # | Gap | Type | Source |
|---|---|---|---|
| 1 | Generalize beyond waitlists (promos, RSVPs, lead capture) | Use-case expansion | AppSumo review by `NicolayLeg` |
| 2 | Bridge to post-launch in-app referrals | Customer journey | Founder's own honest comparison page |
| 3 | Dashboard navigation + missing social fields | UX / polish debt | AppSumo AI summary across all 13 reviews |
| 4 | Full UI translations, not just landing pages | Localization | AppSumo review by `John.doe` |
| 5 | Headless API-first option | Architectural wedge | Reddit r/SideProject, founder of pleasehold.dev |

Counter-evidence ("What's NOT a gap"): pricing is competitive, integrations are generous, AI page builder is the actual differentiator. Don't bet against any of those.

## Step 9 — Output

Two artifacts produced:

- 9-slide Instagram carousel as a single HTML file ([`../examples/waitlister-gaps-carousel.html`](../examples/waitlister-gaps-carousel.html))
- Research report in markdown ([`../examples/waitlister-gaps-report.md`](../examples/waitlister-gaps-report.md))

Both delivered. The user screenshots the carousel slides for IG; the markdown report doubles as a LinkedIn post draft and a YouTube script outline.

---

## Total cost

- Baseline rag-web-browser: ~$0.05
- Reddit attempt 1 (wasted): ~$0.08
- Reddit attempt 2 (scoped): ~$0.06
- X name search: ~$0.01
- X complaint operators: ~$0.01
- Review-site rag-web-browser: ~$0.10

**~$0.31 total** — including the wasted first Reddit pass. With auto-discovery off from the start, would have been ~$0.23.

## Time

~25 minutes wall-clock end to end, including the wasted Reddit pass and the carousel HTML generation.
