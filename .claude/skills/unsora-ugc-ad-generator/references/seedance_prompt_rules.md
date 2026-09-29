# Seedance 2.0 prompt rules

How to write the Seedance 2.0 video prompt for each segment, based on the approved first frame + the chosen ad idea. These prompts drive `create_video` (model `seedance-2.0`).

## Non-negotiables

1. **Never use the word "cinematic."** These are UGC ads meant to look like real iPhone footage.
2. **Seedance 2.0 generates speech/dialogue natively with lipsync.** NEVER tell the user to add voiceover in post — write the dialogue into the prompt's audio direction.
3. **One action arc per prompt.** Don't describe two scene changes in a single 15s prompt.
4. **Be aggressively descriptive** — if you don't describe something, Seedance invents it and you get random artifacts.

## Per-segment prompt structure

Each segment is up to 15s, written as one continuous prompt. Use this shape:

```
9:16. 15 seconds. Single continuous shot. UGC style. iPhone handheld.

@Image1 is the creator. @Image3 is the product.

[0:00-0:05] [Rich, detailed description — 3-4 sentences. Camera position,
the person's full appearance, what they're wearing, their expression, what
their hands are doing, what's on the surface in front of them and what's
NOT there, the specific light source and direction, background details.]

[0:05-0:10] [Rich description — 3-4 sentences. What changes, what the person
does with the product, hand movements, facial expression shift, what's
visible and what's not. Same light source. Background consistent.]

[0:10-0:15] [Rich description — 3-4 sentences. Final movement, expression,
eye contact, body position, product position. Describe the final moment
clearly — this frame matters if the ad continues into another segment.]

Audio: [Voice character — age, gender, tone, energy]. [Room tone matching
the setting]. Natural speech rhythm with pauses. "[Full dialogue with filler
words, contractions, casual pacing]."
```

Adjust the time blocks to the segment length (e.g. a 10s segment = two 5s blocks).

## Anti-cinematic vocabulary

**ALWAYS use:** `iPhone handheld`, `natural lighting` / `window light`, `UGC style`, `slight camera shake`, `casual`, `authentic`, `9:16`.

**NEVER use:** `cinematic`, camera brands (`ARRI`, `RED`, `Blackmagic`), `anamorphic`, `film grain`, `dramatic lighting`, `speed ramp`, `bloom flash`, `lens flare`, `whip pan`, `crane`, `dolly`, `steadicam`, `gimbal`, `Dutch angle`, `color grade`, `LUT`, `bokeh`, `epic`, `breathtaking`, `stunning`, `slow motion` (unless "iPhone slow-mo"), `depth of field` alone (say "phone camera depth of field").

## Detail level (every 5s block needs 3–4 sentences)

- What the person is doing with EACH hand.
- Their exact facial expression.
- What's on the surface and what's NOT there.
- Background details.
- Specific light source and direction.

## Audio direction (required in every prompt)

**Voice** — match the demographic. E.g. "Warm female voice, mid-20s, casual, talking to a friend" or "Deep male voice, 40s, genuine dad energy, not a narrator."

**Room tone — must match the setting:**
- Bathroom: slight reverb from tiled walls.
- Bedroom: soft close acoustics, carpeted, minimal echo.
- Kitchen: open-space feel, subtle ambient sounds.
- Car: muffled close acoustics.
- Outdoors: natural ambience, slight wind.
- Living room: warm room tone, furnished space.

**Speech pattern** — natural, with pauses, filler words, contractions. NOT scripted.

## Dialogue rules — sound real, not scripted

- Use contractions: "I've been," "it's literally," "you're gonna."
- Include fillers: "like," "honestly," "so basically."
- Casual grammar — fragments and run-ons are fine.
- Genuinely excited or skeptical, not rehearsed.

**Good:** "Okay so I've been using this for like two weeks and honestly? It actually works."
**Bad:** "This revolutionary product has transformed my routine completely."

## Reference-image mapping

```
@Image1 = the creator (the approved first frame / creator reference) — SAME across ALL segments
@Image2 = setting reference, if needed
@Image3 = product photo (from the user / product link)
```

Pass these as `referenceImages` on `create_video` so the person and product stay consistent across segments.

## Multi-segment continuity

When the ad is longer than 15s and built as a chain, each segment's Seedance prompt should:
- Continue the story from where the previous segment ended (its last frame is this segment's first frame).
- Keep the same creator, wardrobe, setting, lighting, and room tone described in segment 1.
- Follow the ad arc across segments: Hook → Problem/Proof → Benefit/Demo → CTA.

## Seedance 2.0 facts

- Input: up to 9 images + 3 videos + 3 audio.
- Output: 4–15 seconds per generation, up to 2K, 9:16 for UGC.
- Native audio: dialogue with lipsync, ambient sound, room tone — all generated together.
- Handles long, detailed prompts well, especially with `@Image` references.
