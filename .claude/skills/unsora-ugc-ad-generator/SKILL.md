---
name: unsora-ugc-ad-generator
description: End-to-end UGC video ad generator for general AI agents such as Claude Code, Codex, Claude, OpenClaw, and Hermes, built on the Unsora MCP. Takes a product (link, image, or description), runs deep market research first (20+ searches mining real Reddit posts and comments via the PullPush API plus review sites) to build a Voice-of-Customer brief, then proposes up to 20 UGC ad ideas, each grounded in a research finding and a named copywriting framework (PAS, BAB, AIDA, skeptic's arc, etc.). Builds 1 to 20 finished ads in a single run: for each ad it creates an authentic UGC first-frame image, a Seedance 2.0 video prompt, and the generated video, and can post finished ads to Instagram or TikTok. Ads over 15 seconds are chained (last frame becomes next first frame) and stitched with ffmpeg. Use whenever the user wants a UGC ad, product ad, TikTok/Reel/Shorts ad, a batch of ad variations, or 'an ad for my product,' or gives a product link/photo and asks for ad ideas or videos. Pauses for approval at checkpoints and before any paid generation or post.
---

# Unsora UGC Ad Generator

Runs the full pipeline from a raw product to one or many finished UGC video ads, then optionally posts them. Everything runs through the **Unsora MCP** (`create_image`, `create_video`, `wait_for_*`, `create_post`) plus **ffmpeg** for last-frame extraction and stitching, plus the **PullPush API + web search** for market research. This skill is self-contained — all prompt-writing and research rules live in its own `references/` files.

## Golden rules

1. **Pause at every checkpoint.** This pipeline spends credits. NEVER generate an image or video, and NEVER post, without explicit user approval at the checkpoint right before it. When you pause, stop your turn and wait for the user.
2. **Research before ideas — always.** Never propose ad ideas from imagination. Run the Step 2 market research (≥20 searches: Reddit posts + comments via PullPush, plus review sites via web search) and build the Voice-of-Customer brief first. Every idea must trace to a real finding and a named copywriting framework.
3. **Seedance 2.0 caps at 15 seconds per generation.** Anything longer is built as a chain of ≤15s segments and stitched. See "Step 8 — Duration branching."
4. **Keep the creator and product consistent** across every segment by reusing the same reference image(s).
5. **Read the matching reference file before each research/writing/generation step** — they hold the exact rules and parameters.
6. **Batch is the default shape, single is just batch of one.** The user can generate anywhere from 1 to 20 ads in a single run. When generating more than one, **consolidate checkpoints** — approve the whole set at each stage rather than pausing per ad (20 ads × 6 pauses would be unusable). See "Batch mode."

Set up a workspace once: `mkdir -p /home/claude/unsora-ugc-ad-generator/work/research`. In batch runs, give each ad its own subfolder: `work/ad01/`, `work/ad02/`, … so frames, segments, and finals never collide. Research artifacts (raw API responses, VoC brief) live in `work/research/`.

## Batch mode (1–20 ads)

The user chooses how many ads to generate (1–20). One ad = run the steps below straightforwardly. Two or more = run the **same** steps but batched, with these differences:

- **One product intake** (Step 1), **one research pass** (Step 2), and **one idea list** (Step 3) serve the whole batch.
- At Step 4 the user selects **which ideas** to build (e.g. "make 5: numbers 1, 3, 7, 12, 18" or "generate all 20") and gives a duration + ratio that applies to all (or per-idea if they specify).
- **Consolidate each checkpoint across the batch.** Instead of pausing once per ad per stage:
  - **Batch checkpoint 2** — write ALL first-frame prompts, present them as a numbered set, get one approval (user can approve all or flag specific ones to change).
  - **Batch checkpoint 3** — generate ALL first frames, present them together, get one approval (user flags any to regenerate).
  - **Batch checkpoint 4** — write ALL segment-1 Seedance prompts, present as a set, get one approval to generate videos.
- **Generate in a loop, report as a batch.** Generate the frames/videos for every selected ad, tracking each by its ad number + workspace subfolder. If one fails, keep going with the rest and report failures at the end — don't halt the whole batch.
- **Multi-segment ads inside a batch:** each ad that exceeds 15s still chains + stitches per Step 8B. Do the chaining per ad; you may batch the per-segment approval across ads that are on the same segment index to keep pauses manageable.
- **Track everything in a manifest** so nothing gets lost across 20 ads — see "Batch manifest."

### Batch manifest

For multi-ad runs, keep a simple table in the conversation (and optionally `work/manifest.json`) with, per ad: number, idea title, framework, duration, segment count, first-frame status, video status, final file path, post status. Update and re-show it as stages complete so the user always sees where all N ads stand.

---

## Step 1 — Intake the product

Accept the product in whatever form the user gives:

- **Link** → fetch the page with `web_fetch`. Pull the product name, what it does, key selling points, target audience, and the main product image URL. Download that image to the workspace for use as a reference later.
- **Image (uploaded)** → look at it. If the image alone doesn't tell you what the product is, what it's called, and who it's for, **ask the user** for those details before continuing. Don't invent a brand or benefit that isn't visible.
- **Text description / prompt** → use it directly.
- **Combination** (image + description, link + notes, etc.) → merge everything.

By the end of Step 1 you must know: product name, what it does, 2–4 key benefits, target audience, whether you have a usable product image, **and the market/niche this product lives in** (needed to pick subreddits and search terms in Step 2). Ask for anything missing that matters, then confirm your understanding back in 2–3 lines.

## Step 2 — Deep market research (before any ideas)

Read `references/market_research_rules.md` and follow it exactly. In short:

1. From the Step 1 intake, name the **market/niche** and identify **3–6 target subreddits** where this product's buyers actually hang out.
2. Run **at least 20 distinct research queries**: ~12–14 against the **PullPush API** (`api.pullpush.io` — the verified way to read real Reddit posts AND comments from this environment; reddit.com itself is egress-blocked, so never waste time on it) covering pains, objections ("waste of money", "actually work", "regret"), failed alternatives, competitor sentiment, and full comment threads of the highest-value posts — plus ~6–8 `web_search` queries across review roundups, Trustpilot/complaint pages, and "{category} reddit" articles for recency.
3. Save raw responses to `work/research/` and distill everything into a **Voice-of-Customer brief**: top 5 pains with verbatim quotes, objections/skeptic language, failed alternatives, desired-outcome language, trigger moments, recurring phrases, competitor sentiment. Save it to `work/research/voc_brief.md` and show it to the user.

No checkpoint pause is required here (research spends no credits), but narrate progress and present the VoC brief before moving on. Never fabricate a finding — everything in the brief must trace to an actual search result.

## Step 3 — Propose up to 20 research-grounded UGC ad ideas

Output a numbered list of **20** distinct UGC ad concepts (if the user has already said they want only a few, you can offer fewer, but 20 is the default menu so they have range to pick from). Vary the **hook angle**, **scenario**, **creator type**, and **vibe** — not twenty flavors of one idea. Spread across: problem/solution, before/after, "things I wish I knew," skeptic-turned-believer, day-in-the-life, unboxing reaction, comparison, testimonial, tutorial/demo, trend-jacked hook, POV, green-screen react, street interview, "get ready with me," myth-busting, etc.

**Every idea must be grounded in the Step 2 research.** Each idea gives:

- a short **title**,
- the **hook** (first spoken line or on-screen text) — built from real customer phrasing found in research wherever possible,
- the **creator + setting** in a phrase,
- the **copywriting framework** it follows (PAS, BAB, AIDA, Star–Story–Solution, skeptic's QUEST arc, Us-vs-Them, social-proof stack, 4Ps, myth-bust — per `references/market_research_rules.md`),
- the **research finding** it's built on, in a few words (e.g. "pain #2: 'dorky under clothes' complaints from r/Posture").

An idea that can't point to a finding + framework doesn't make the list.

**► CHECKPOINT 1 — stop here.** End with: "Tell me **which** ideas to build and **how many** (any number from 1 to 20 — e.g. 'just #4', 'numbers 1, 3, 7, 12, 18', or 'generate all 20'), or give me your own idea(s). Also let me know the total length per ad (e.g. 15s, 30s) and the aspect ratio (default 9:16 vertical)." Wait for the user.

## Step 4 — Lock the selection

The user selects which ideas to build (1–20) or supplies their own. Confirm, for the batch:
- the **list of ads** to build (idea title + hook + framework + creator/setting for each),
- the **total duration** per ad (shared, unless they specified per-ad),
- the **aspect ratio** (default 9:16),
- the **segment count** per ad = `ceil(total_seconds / 15)`. State it plainly, e.g. "5 ads, each 30s = 2 segments each = 10 generations total."
- If more than one ad: create the workspace subfolders (`work/ad01/`…) and start the **batch manifest**.

From here, if only one ad was selected, run Steps 5–9 straight through. If more than one, run Steps 5–8 in **batch mode** (consolidated checkpoints — see "Batch mode" above), then Step 9 for posting.

## Step 5 — Write the UGC first-frame prompt

Read `references/first_frame_prompt_rules.md` and follow it to write the first-frame prompt for the chosen idea: cover its eight components, apply 4–5 anti-AI-default moves, and lock the cultural context so the frame visibly matches the product's audience (use the Step 2 research to make the creator, setting, and props ring true for this niche). Deliver the prompt between `---` delimiters.

**Batch:** write a first-frame prompt for **each** selected ad, labeled by ad number, and present them as one numbered set.

**► CHECKPOINT 2 — stop here.** Single: "Good to generate the first frame with this? (or tell me what to change)." Batch: "Here are all N first-frame prompts. Approve the set to generate, or tell me which numbers to change." Wait for approval.

## Step 6 — Generate the first frame

Read `references/unsora_and_ffmpeg.md` for exact parameters. On approval, call `Unsora AI:create_image` (prompt, `aspectRatio`, `model` default `nano-banana-2`, `referenceImages` = product image URL(s)), then poll with `Unsora AI:wait_for_image`. Download the result to the workspace and show it.

**Batch:** loop over every approved prompt, generating each ad's first frame into its own `work/adNN/` subfolder. Update the manifest as each completes. Present all frames together.

**► CHECKPOINT 3 — stop here.** Single: "Here's the first frame. Approve it, or want a regeneration / tweak?" Batch: "Here are all N first frames. Approve the set, or tell me which numbers to regenerate." Adjust and repeat for any flagged. Only proceed once approved — these frames anchor the videos.

## Step 7 — Write the Seedance 2.0 prompt (segment 1)

Read `references/seedance_prompt_rules.md` and follow it to write the Seedance 2.0 prompt for **segment 1**, based on the approved first frame + the chosen idea. Follow all its rules (no "cinematic," native audio/dialogue with lipsync, dense per-block detail, `@Image` mapping). Keep the spoken lines in the register the research surfaced — real customer language, in the shape of the idea's framework.

- **One segment total** → write just this one prompt now.
- **Multi-segment** → write only segment 1 now; later segments are written one at a time during the chain (Step 8B), because each depends on the actual last frame of the previous generated video.

**Batch:** write the segment-1 Seedance prompt for **each** selected ad, labeled by ad number, and present them as one set.

**► CHECKPOINT 4 — stop here.** Single: "Approve this to generate the video? (heads up: this spends credits)." Batch: "Here are all N segment-1 prompts. Approve the set to generate videos (this spends credits for N generations), or flag any to change." Wait for approval.

## Step 8 — Generate the video (branch on duration)

Read `references/unsora_and_ffmpeg.md` before running anything here. **Batch:** run this step for every selected ad, each in its own `work/adNN/` subfolder, updating the manifest as each finishes. If an ad fails, note it and continue with the rest; report all failures at the end.

### Case A — total ≤ 15s (single segment)

1. `Unsora AI:create_video` with `model: "seedance-2.0"`, the approved prompt, `image` = approved first frame, `aspectRatio`, `duration` = total seconds, audio on.
2. Poll with `Unsora AI:wait_for_video`; download the MP4.
3. Present the finished video (batch: collect it; present all at the end). Then go to **Step 9**.

### Case B — total > 15s (chained segments)

Loop for each segment `i` from 1 to N:

1. **Generate segment i** with `create_video` (Seedance 2.0), using `image` = the first frame for this segment (segment 1 = approved Step 6 frame; segment i>1 = extracted last frame of segment i−1), the Seedance prompt for segment i, and `duration` ≤ 15s. Pass the product photo (and creator reference where the `@Image` mapping calls for it) in `referenceImages` to prevent drift.
2. Poll with `wait_for_video`; download the segment MP4.
3. **If this is the last segment, stop the loop.** Otherwise:
   - **Extract the last frame** of segment i with ffmpeg → PNG. This becomes the **first frame of segment i+1**.
   - **Write the Seedance prompt for segment i+1** (per `references/seedance_prompt_rules.md`), continuing the ad's arc (Hook → Problem/Proof → Benefit/Demo → CTA) from where segment i ended, anchored on the extracted frame.
   - **► CHECKPOINT (per segment) — stop here.** Show the extracted last frame + the next segment's prompt and get approval before generating it.
4. After all segments are generated, **stitch** them in order with ffmpeg into `final_ad.mp4` (batch: `work/adNN/final_ad.mp4`).
5. Present the final stitched video (batch: collect it). Then go to **Step 9**.

**Batch presentation:** once every selected ad is finished, present all finals together with `present_files` and show the completed manifest (each ad's title, framework, duration, and file), noting any that failed.

## Step 9 — Optional: post to Instagram / TikTok

Ask the user if they'd like to post the finished ad(s) now. **Batch:** they can post all, some, or none — confirm which ads go to which accounts, and whether to post now or schedule (e.g. stagger over days). Draft a caption per ad. If yes, follow the posting flow in `references/unsora_and_ffmpeg.md`:

1. `get_subscription` → confirm `isActive` (posting needs a paid plan). If inactive, say so and stop.
2. `get_accounts` → pick the IG/TikTok `accountIds` to post to.
3. Make sure the final MP4 is reachable at a public URL for `mediaUrl` (ask the user how to host it if only a local file exists).
4. Draft a **caption** from the ad idea and let the user edit it.

**► CHECKPOINT (before posting) — stop here.** Show the target account(s), caption, and whether it's post-now or scheduled. Only on approval call `Unsora AI:create_post` (`accountIds`, `caption`, `mediaType: "video"`, `mediaUrl`, optional `scheduled_at`). Confirm what posted where.

---

## Checkpoint summary (never skip these)

In batch runs, each checkpoint below is **consolidated** — one approval covers the whole set, not one pause per ad. (Step 2 research has no checkpoint — it spends no credits — but the VoC brief is always shown before ideas.)

1. After the research-grounded ideas → user picks which ideas (1–20) + gives duration/ratio.
2. After the first-frame prompt(s) → approve before generating the image(s).
3. After the first-frame image(s) → approve before writing the video prompt(s).
4. After the segment-1 Seedance prompt(s) → approve before generating video(s).
5. (Multi-segment only) After each extracted last frame + next prompt → approve before generating the next segment.
6. (If posting) Before `create_post` → approve accounts + caption + timing.

## When things go wrong

- **PullPush returns 5xx / is down** → fall back to `web_search`-only research, tell the user Reddit depth is reduced this run, and still hit ≥20 total queries.
- **Generation returns FAILED** → tell the user, show any error, offer to retry or adjust. Don't silently loop.
- **Requires paid plan / no credits** → check `get_subscription` (`isActive`) and `get_credits`, relay what's needed.
- **Product image won't load from a link** → ask the user to upload the product photo directly.
- **Blurry extracted last frame** → extract a slightly earlier frame (see the ffmpeg reference).
- **Posting needs a public URL** → the connector reads `mediaUrl`; if you only have a local file, ask the user how they want to host it.

## Reference files

- `references/market_research_rules.md` — verified PullPush API endpoints, the ≥20-query research recipe, VoC brief format, and the copywriting-framework grounding rules. Read before Step 2.
- `references/first_frame_prompt_rules.md` — how to write the UGC first-frame image prompt. Read before Step 5.
- `references/seedance_prompt_rules.md` — how to write each Seedance 2.0 video prompt. Read before Step 7 and each chained segment.
- `references/unsora_and_ffmpeg.md` — exact Unsora tool parameters, polling, the ffmpeg extract/stitch commands, and the posting flow. Read before Steps 6, 8, and 9.
