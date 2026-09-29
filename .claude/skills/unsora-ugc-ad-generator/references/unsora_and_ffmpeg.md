# Unsora MCP + ffmpeg reference

Exact tool parameters, polling patterns, and verified ffmpeg commands. Read before the generation and stitching steps.

All Unsora generation tools are **async**: `create_*` queues a job and returns a `generation.id`; poll with the matching `wait_for_*` until `COMPLETED` (or `FAILED`). Download the media to the workspace before using or presenting it.

---

## Pre-flight (only when diagnosing a failure)

- `Unsora AI:get_subscription` → check `isActive` before posting / heavy generation.
- `Unsora AI:get_credits` → current balance.
- `Unsora AI:get_accounts` → connected IG/TikTok accounts (needed only for posting).

Don't call these every run — use them when a call is rejected ("requires paid plan", "insufficient credits").

---

## Generate the first frame — `Unsora AI:create_image`

- `prompt` (required) — the approved UGC first-frame prompt.
- `aspectRatio` — `"9:16"`, `"1:1"`, `"16:9"`.
- `model` — `nano-banana-2` (default), `nano-banana-pro` (higher quality), `seedream-v5-lite`, `gpt-image-1.5`, `gpt-image-2`.
- `referenceImages` — array of URLs; pass the product image so the real product appears in frame.
- `resolution` — optional.

Then **`Unsora AI:wait_for_image`** — `generationId` (required), optional `intervalSeconds` (2–60), `maxAttempts` (1–180). On COMPLETED, download the image:

```bash
curl -L -o /home/claude/unsora-ugc-ad-generator/work/first_frame.png "<IMAGE_URL>"
```

---

## Generate a video segment — `Unsora AI:create_video`

- `prompt` (required) — the approved Seedance prompt for this segment.
- `model` — `"seedance-2.0"` (or `"seedance-2.0-fast"`). Stick with Seedance 2.0: the prompts are written for it and it does native audio + lipsync.
- `image` — the first frame for this segment (segment 1 = approved first frame; segment i>1 = extracted last frame of segment i−1).
- `aspectRatio` — e.g. `"9:16"`.
- `duration` — integer seconds, **max 15** per generation.
- `referenceImages` — up to 9 URLs; pass the product photo (and the creator reference when `@Image` mapping calls for it) to hold consistency across segments.
- `generateAudio` / `sound` — keep audio on so dialogue + room tone render.
- `lastImage` — optional explicit end frame; usually leave unset (continuity is driven by extracting the last frame).

Then **`Unsora AI:wait_for_video`** — `generationId` (required), optional `intervalSeconds` (2–60), `maxAttempts` (1–180). On COMPLETED, download the segment:

```bash
curl -L -o /home/claude/unsora-ugc-ad-generator/work/seg1.mp4 "<VIDEO_URL>"
```

---

## ffmpeg — extract the last frame (chain step)

The final frame of a segment becomes the first frame (`image`) of the next segment. Verified with ffmpeg 6.1.

```bash
# Last frame → PNG
ffmpeg -y -sseof -0.1 -i seg1.mp4 -update 1 -q:v 2 seg1_lastframe.png

# If the final frame is motion-blurred, pull ~0.2s earlier instead:
ffmpeg -y -sseof -0.2 -i seg1.mp4 -vframes 1 -q:v 2 seg1_lastframe.png
```

If Unsora's `image` needs a URL rather than a local path, host/reference the PNG per the connector's expectation.

---

## ffmpeg — stitch segments into the final video

Preferred: fast concat with stream copy (works when segments share codec/resolution/fps — they will, being the same Seedance model + ratio). Verified with ffmpeg 6.1.

```bash
printf "file 'seg1.mp4'\nfile 'seg2.mp4'\nfile 'seg3.mp4'\n" > concat_list.txt
ffmpeg -y -f concat -safe 0 -i concat_list.txt -c copy final_ad.mp4
```

Fallback if `-c copy` errors (codec/param mismatch):

```bash
ffmpeg -y -f concat -safe 0 -i concat_list.txt \
  -c:v libx264 -pix_fmt yuv420p -c:a aac -movflags +faststart final_ad.mp4
```

Verify duration:

```bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 final_ad.mp4
```

---

## Optional final step — post to Instagram / TikTok

Only after the user opts in. Flow:

1. `Unsora AI:get_subscription` → confirm `isActive` (posting requires a paid plan). If inactive, tell the user and stop.
2. `Unsora AI:get_accounts` → list connected IG/TikTok accounts; get the `accountIds` the user wants to post to.
3. Host/upload the final MP4 so it has a public URL Unsora can read (the connector needs a `mediaUrl`). If you only have a local file, tell the user Unsora needs a reachable URL and ask how they'd like to host it.
4. `Unsora AI:create_post`:
   - `accountIds` (required) — array of chosen account ids.
   - `caption` (required) — draft a caption from the ad idea; let the user edit before posting.
   - `mediaType: "video"`.
   - `mediaUrl` — the final ad URL.
   - `scheduled_at` — optional ISO timestamp for scheduling; omit to post now.
5. Confirm back what posted where (or when it's scheduled). You can verify with `Unsora AI:list_posts`.

For a **slideshow** post instead of a video, use `mediaType: "slideshow"` with `mediaUrls` (array of image URLs). `Unsora AI:create_image_and_schedule` can generate an image and schedule a slideshow in one call.

---

## Quick reference — full asset flow

```
product → create_image (+wait_for_image) → first_frame.png
first_frame.png + seedance prompt → create_video (+wait_for_video) → seg1.mp4
  ├─ total ≤15s → seg1.mp4 IS the final ad
  └─ total >15s → extract last frame → seg2 first frame
                  → next seedance prompt → create_video → seg2.mp4
                  → (repeat per segment) → concat seg1..segN → final_ad.mp4
final_ad.mp4 → (optional) create_post → Instagram / TikTok
```
