# Apify actors cheatsheet — market-gap-research

Read this before your first `call-actor` of a run. Pick the cheapest viable actor that returns the content you need, not just metadata.

---

## `clearpath/reddit-search-scraper`

Highest-signal source for SaaS product complaints. Returns posts AND comments.

**Pricing:** $0.00099 per result.

**Input fields that matter:**

| Field | Type | Notes |
|---|---|---|
| `query` | string (max 700 chars) | Supports boolean operators: `OR`, `AND`, quoted phrases |
| `maxResults` | int | `0` = unlimited. Use `60–100` for a first pass. |
| `contentType` | enum | `"posts"`, `"comments"`, or `"both"`. Use `"both"`. |
| `sort` | enum | `"relevance"`, `"new"`, `"top"`, `"hot"`, `"comments"`. Use `"relevance"` for gap research. |
| `subreddits` | array of strings | Explicit subreddit allow-list, e.g. `["SaaS", "indiehackers"]`. **Required** for products with generic names. |
| `autoDiscoverSubreddits` | bool | Default `true`. **Set `false`** for generic-named products (waitlist, stack, pulse, loop). |
| `maxSubreddits` | int | Default 20. Rarely needs changing. |

**Recommended call for SaaS gap research:**

```json
{
  "query": "<product> OR <product variant>",
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

**Cost estimate:** 60 results × $0.00099 ≈ $0.06.

---

## `simoit/x-twitter-search-tweets-scrapper`

Cheapest viable X scraper. Use for both name-only searches AND comparison/complaint operator searches.

**Pricing:** $0.0003 per result. Very cheap.

**Input fields that matter:**

| Field | Type | Notes |
|---|---|---|
| `searchTerms` | string | Comma- or newline-separated for multiple terms. Supports `OR`, quoted phrases. |
| `sort` | enum | `"latest"` or `"top"`. Use `"top"` for gap research (filters spam). |
| `includeReplies` | bool | Default `true`. Keep `true` — replies often have the actual complaint. |
| `includeRetweets` | bool | Default `true`. Set `false` if too noisy. |
| `maxItems` | int | Hard cap. Use `20–40` per search. |
| `maxPages` | int | Page-level cap. Set `2–3`. |
| `limit` | int | Per-page cap. Default fine. |

**Recommended pair of calls (name search + complaint search):**

Call 1 — name only:

```json
{
  "searchTerms": "<product>",
  "sort": "top",
  "includeReplies": true,
  "includeRetweets": false,
  "maxItems": 30
}
```

Call 2 — complaint operators:

```json
{
  "searchTerms": "\"<product> alternative\" OR \"<product> vs\" OR \"better than <product>\" OR \"switching from <product>\"",
  "sort": "top",
  "includeReplies": true,
  "includeRetweets": false,
  "maxItems": 20
}
```

**Cost estimate:** 50 total tweets × $0.0003 ≈ $0.02.

---

## `apify/rag-web-browser`

Already loaded as `Apify:apify--rag-web-browser` MCP tool. Use for review sites, founder pages, comparison articles. Returns markdown-formatted content from Google + page fetch.

**Pricing:** per query (varies). Cheaper than running a dedicated scraper for one-off queries.

**Input fields:**

| Field | Notes |
|---|---|
| `query` | Either a Google search query OR a specific URL |
| `maxResults` | Use `3–5` |
| `outputFormats` | Array. Default `["markdown"]` — keep this. |

**Recommended queries (run all three for a thorough sweep):**

1. `"<product> review limitations cons missing features"` — surfaces AppSumo, G2, Capterra
2. `"<product> alternatives comparison"` — surfaces blog posts ranking competitors
3. `"<product> honest review"` — surfaces YouTube creator reviews and Reddit threads the scoped Reddit search may have missed

If the product has a strong category presence, also try:

4. `"<product>" site:reddit.com` — catches Reddit threads outside your subreddit list
5. `"<product> vs <known competitor>"` — surfaces direct comparison pages

---

## `apify/instagram-search-scraper`

**Default: skip this.** Use only when the user explicitly asks for Instagram data, OR the product is consumer-facing with strong IG presence.

Returns hashtag/user/place **metadata only** — not comment-level discussion. Burning credits on it for B2B SaaS produces no signal.

**Input fields:**

| Field | Notes |
|---|---|
| `search` | Keyword |
| `searchType` | `"place"`, `"user"`, or `"hashtag"` |
| `searchLimit` | Int |

If you do run it, pair with `apify/instagram-comment-scraper` afterward on the highest-engagement posts to actually surface complaints. That second call is what costs real money — budget accordingly.

---

## How to pick a new actor (when defaults don't fit)

If you need a platform not listed above (e.g. TikTok, ProductHunt, HackerNews, Discord):

1. `Apify:search-actors` with **just the platform name** (e.g. `"hackernews"`, not `"hackernews scraper"`).
2. Run **at least two searches** with different keyword phrasings to compare.
3. Sort candidates by:
   - **Price per result** (lowest wins, but watch for tiered pricing — the first 100 results free then $X/result is common)
   - **Monthly user count** (proxy for reliability)
   - **Stars / success rate** if shown
4. Use `Apify:fetch-actor-details` on top 2 candidates to read the input schema.
5. Pick the one whose schema returns **post content + comments**, not just metadata.

If you can't find one that returns comments, fall back to `apify/rag-web-browser` with a `site:<platform>.com` query — slower but always works.

---

## Cost discipline

A typical full run should cost **< $0.50**:

| Source | Calls | Approx cost |
|---|---|---|
| rag-web-browser baseline | 1 | ~$0.05 |
| Reddit | 1 (60 results) | ~$0.06 |
| X name search | 1 (30 tweets) | ~$0.01 |
| X complaint operators | 1 (20 tweets) | ~$0.01 |
| rag-web-browser review sites | 2 (5 each) | ~$0.20 |
| **Total** | **6 calls** | **~$0.33** |

If a run is creeping past $1, **stop and re-scope**. The user will notice the credit burn and would rather refine queries than pay for noise.
