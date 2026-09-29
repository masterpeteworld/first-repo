# UGC first-frame prompt rules

How to write the opening-frame prompt for a UGC ad. The output is a single dense prose paragraph (or a few) that an image model reads as one continuous instruction. Deliver it between `---` delimiters.

## Core philosophy — beat the AI-default look

When a model isn't given specific direction it produces sterile output: symmetrical clean rooms, flawless skin, generic "studio/cafe" backgrounds, even lighting, Western defaults. Fight that on three axes, using several moves per prompt:

1. **Imperfection** — wrinkles, stains, asymmetry, wear, clutter, things slightly off.
2. **Specificity** — named brands, exact colors, particular textures, specific cultural items.
3. **Camera honesty** — phone-camera artifacts, lens distortion, color cast, real lighting conditions.

If the prompt could describe *any* room, *any* person, *any* setup, it isn't specific enough. The frame should feel like it could only be that one place, that one person, that one moment.

## Eight required components (cover all of them, in this order)

1. **Camera setup** — device implied (iPhone front cam, DSLR, GoPro), angle, height, distance, format (9:16 vertical for Reels/TikTok, 16:9 for podcasts/YouTube), lens characteristics (wide-angle distortion, depth of field).
2. **Subject's body and expression** — age, ethnicity when relevant, build, hair (style + texture + condition), facial expression mid-action, eye direction, posture. Aim for "caught mid-something," not "posed."
3. **Outfit details** — specific garments, specific colors, fit (oversized/cropped/wrinkled), realistic brand cues, layering, accessories. Include at least one imperfection (coffee stain, frayed hem, uneven drawstrings).
4. **Setting** — a space with character: specific furniture, cultural markers, the right clutter for that person's life. Avoid "modern minimalist" unless that's the point.
5. **Background props / environmental details** — what's on the desk, walls, floor. List 5–10 specific items; these sell authenticity.
6. **Lighting** — source (window, ring light, mixed, harsh fluorescent, golden hour), direction, color-temperature mismatch when relevant, shadows, an imperfection in the lighting.
7. **Texture and material callouts** — skin texture, fabric weave, surface materials. Tells the model to render fine detail instead of smoothing.
8. **Color grading and camera artifacts** — slight chromatic aberration, motion blur, white-balance imperfection, grain, the specific color tone of phone footage in mixed light.

## Anti-AI-default toolkit (use 4–5 per prompt)

- **Asymmetry** — "one armrest lower than the other," "drawstrings pulled to different lengths," "string lights sagging more on the left."
- **Wear and tear** — "small coffee stain near the collar," "cracking on the armrest," "phone case cracked in one corner," "pharmacy calendar hanging crooked on the wrong month."
- **Organic clutter** — "charging cable trailing off-frame," "a crumpled energy-drink can pushed aside," "tangled earphones," "sandals kicked off near the door."
- **Off-default colors** — amber "ON AIR" with one letter dimmer instead of red neon; "cream walls with paint cracks near the ceiling" instead of clean white.
- **Brand specificity** — real, mid-tier, regionally-correct brands beat generic descriptions. Bangladesh: Walton fan, Bata chappals, Pran/Mr. Twist snacks, Robi/Grameenphone merch. American casual: Patagonia quarter-zip, Hydroflask. Desk: "IKEA Karlby on Alex drawers."
- **Lighting imperfection** — "ring light off-center casting uneven illumination," "window light from the right creating a warm color-temperature mismatch," "harsh afternoon sun mixed with yellow indoor tube light."
- **Camera realness** — "slight motion blur typical of video stills," "chromatic aberration on high-contrast edges," "authentic front-camera iPhone quality with visible pores," "fingerprints on the mirror."

## Cultural specificity

If a country/region/context is given, the frame must visibly be that context — not generic "Asian apartment." Lean on details a local would recognize:

- **Bangladesh** — Walton appliances, Bata sandals, Pran/Mr. Twist snacks, Robi/Grameenphone merch, mismatched plastic stackable chairs, local-pharmacy calendar, Dhaka University tees, casual salwar kameez, mismatched tile floors, oversaturated yellow-green phone grade.
- **American college** — dorm fairy lights, polaroid bulletin boards, branded hoodies (Champion, Nike), Hydroflask, sticker-covered MacBook.
- **Tech / podcast** — Patagonia/Carhartt, IKEA desk hardware, USB mics on boom arms (Blue Yeti, Rode PodMic), Funko Pop, neon sign, exposed XLR cables.

If the product/audience implies a context, use it. If it's genuinely ambiguous and would change the output a lot, ask.

## Output rules

- One or more dense prose paragraphs — **no bullet points inside the actual prompt**.
- Length: 200–500 words for a single character/setting; 500–800 for multi-person or richly dressed sets.
- Lead with camera setup → subject → outfit → setting → props → lighting → texture → color/artifacts.
- **Only describe clean scenes** — no emoji stickers, watermarks, text overlays, or app UI, because the generator recreates everything it "sees" as a physical object.
- Deliver between `---` delimiters. Optionally add one line naming the authenticity moves used so the user can tweak.

## Calibration example (solo creator, lifestyle UGC)

Intent: "First frame of a UGC video for AG1, college dorm, 18-year-old wasian girl in bikini at study desk, hair in bun, AG1 pouch in frame but not used yet."

> A close-up, intimate iPhone front-facing camera video still captures an 18-year-old wasian young woman from chest-up, positioned directly in front of her study desk, leaning slightly toward the camera with an authentic, conversational expression mid-sentence. Her hair is pulled back into a casual high messy bun with a few loose strands framing her face naturally. She wears a simple solid-colored bikini top, appearing relaxed and comfortable in her personal space. Her face shows genuine animation as if she's just started speaking to a friend, with slightly raised eyebrows and an engaged "wait till you hear this" expression, looking directly into the camera lens creating intimate eye contact typical of authentic UGC content. The cozy dorm room study table setting is visible in the shallow depth of field background, featuring scattered textbooks, a laptop partially visible, fairy lights strung along a bulletin board with polaroid photos pinned up, and a clear glass of water on the desk. An AG1 green pouch is casually positioned on the desk within the frame but not being interacted with yet. Soft natural morning sunlight streams through a nearby window creating gentle, flattering illumination with realistic skin texture, natural shadows under her collarbones, and warm golden tones. The composition captures the characteristic vertical 9:16 TikTok/Reel format with slight motion blur typical of video stills, authentic front-camera iPhone quality with natural skin texture, visible pores, and the genuine, unpolished aesthetic of real creator content filmed in natural bedroom lighting.
