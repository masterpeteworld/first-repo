# Market Research Rules — Reddit Voice-of-Customer + Review Mining

Read this before Step 2 (Deep market research). The goal: before proposing a single ad idea, know how real people in this niche talk — their pains, objections, failed alternatives, desired outcomes, and exact phrases. Ad ideas written without this read like ads. Ad ideas written from this read like posts.

## How to access Reddit (verified, environment-specific)

**Do NOT try reddit.com, old.reddit.com, or api.reddit.com from bash — they are hostname-blocked by the egress proxy.** Do NOT use redlib/safereddit mirrors — they run Anubis bot-checks, and bypassing bot detection is prohibited. Do NOT expect `web_search` to return reddit.com URLs — the search index only returns third-party articles about Reddit sentiment (still usable as secondary sources).

**Use the PullPush API instead** (`api.pullpush.io`) — a public Reddit research archive, no login, returns real post bodies and real comment text. Verified working endpoints:

```bash
UA="Mozilla/5.0"

# 1. Search posts (submissions) by keyword, optionally within a subreddit, sorted by score
curl -s -A "$UA" "https://api.pullpush.io/reddit/search/submission/?q=QUERY&size=25&sort_type=score&sort=desc"
curl -s -A "$UA" "https://api.pullpush.io/reddit/search/submission/?q=QUERY&subreddit=SUBNAME&size=25&sort_type=score&sort=desc"

# 2. Search COMMENTS directly by keyword — the goldmine for verbatim customer language
curl -s -A "$UA" "https://api.pullpush.io/reddit/search/comment/?q=QUERY&size=50"
curl -s -A "$UA" "https://api.pullpush.io/reddit/search/comment/?q=QUERY&subreddit=SUBNAME&size=50"

# 3. Pull the full comment thread of a specific post (use the post's `id` field)
curl -s -A "$UA" "https://api.pullpush.io/reddit/search/comment/?link_id=POST_ID&size=100"
```

Useful JSON fields — submissions: `title`, `selftext`, `subreddit`, `score`, `num_comments`, `id`, `created_utc`. Comments: `body`, `subreddit`, `score`, `link_id`.

Parse with python3/`jq`, and save every raw response into the workspace (`work/research/`) so nothing is lost. URL-encode queries (spaces → `+`).

**Caveats:** PullPush is an archive — the most recent weeks can be spotty, and comment threads are sometimes incomplete. Compensate with `web_search` for recency (see below). If PullPush is down (5xx), fall back to `web_search`-only research and tell the user Reddit depth is reduced this run.

## The minimum-20-searches rule

Run **at least 20 distinct research queries** per product before writing ideas. Default split:

**A. PullPush — ~12–14 queries.** Cover these angles (substitute the product/niche terms):

1. `{product category}` — post search, global, by score (what dominates the conversation)
2. `{product category}` — post search inside each of the 3–6 target subreddits identified in intake
3. `{product category} recommend` — comment search (what people tell each other to buy)
4. `{product category} actually work` — comment search (skepticism language)
5. `{product category} waste of money` — comment search (objections, verbatim)
6. `{product category} regret` OR `returned` — comment search (post-purchase disappointment)
7. `{problem the product solves}` — post search (how people describe the pain when no product is mentioned — best hook material)
8. `{problem} tried everything` — comment search (failed-alternatives stories)
9. `alternative to {well-known competitor}` — comment search
10. `{competitor name}` — post search (what people love/hate about the incumbent)
11. `best {product category}` — post search sorted by num_comments (recommendation threads)
12. Pull full comment threads (`link_id`) for the 2–4 highest-value posts found above
13.–14. Follow-up queries on whatever surprising theme emerged (always leave room to chase a thread)

**B. web_search — ~6–8 queries.** For recency and non-Reddit voices:

- `{product category} reddit` (SEO roundups of Reddit sentiment — secondary source, cite as such)
- `{product category} review 2026` / `honest review`
- `{competitor} trustpilot` or `{competitor} complaints`
- `{product category} before and after`
- `is {product category} worth it`
- niche forums / Quora / YouTube-review summaries for this space

Adjust the mix to the niche (e.g., beauty → add r/SkincareAddiction-style subs; SaaS → add r/Entrepreneur, G2/Capterra searches). More than 20 is fine; fewer is not. Skip a Checkpoint pause here — research spends no credits — but show progress as you go.

## Distill into a Voice-of-Customer (VoC) brief

After the searches, write a compact brief in the conversation (and save to `work/research/voc_brief.md`):

1. **Top 5 pains** — each with 1–2 **verbatim quotes** (short, reworded only if long) + which subreddit/source
2. **Top objections & skepticism** — the exact "scam / gimmick / doesn't work / too expensive" language
3. **Failed alternatives** — what people tried before and why it failed them
4. **Desired outcome language** — how people describe success in their own words
5. **Trigger moments** — the situations that make people go looking (e.g., "my back after 8h at a desk")
6. **Recurring phrases / slang / community in-jokes** worth stealing for hooks
7. **Competitor sentiment** — what the incumbent gets praised and roasted for

Every item must trace back to something actually found in the research — never invent a "finding."

## Grounding ideas in copywriting frameworks

Each of the 20 ad ideas (next step) must name its framework and the research finding it's built on. Frameworks to spread across:

- **PAS** (Problem–Agitate–Solve) — lead with a top-5 pain, agitate with the verbatim worst-case, resolve with the product
- **BAB** (Before–After–Bridge) — before = trigger moment from research; after = desired-outcome language
- **AIDA** — attention hook stolen from a high-score post title
- **Star–Story–Solution** — creator retells a real failed-alternatives arc found in comments
- **Skeptic's arc (QUEST)** — open with the strongest objection found ("I thought these were a scam…"), qualify, then convert
- **Us-vs-Them** — position against the roasted incumbent using its actual complaint language
- **Social-proof stack** — "the entire r/X community keeps recommending…" style (only if research actually shows that)
- **4Ps** (Promise–Picture–Proof–Push) — promise phrased in desired-outcome language; proof from demo
- **Myth-bust** — take a wrong belief that appears repeatedly in comments and correct it

Hooks should reuse **real customer phrasing** (lightly adapted, never fabricated quotes attributed to real users). An idea that can't point to a finding + framework doesn't make the list.
