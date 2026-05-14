# Higgsfield Photo Generation — Reference

---

## When this file is loaded

This file is loaded in Phase 2 — AFTER the user has confirmed landing is complete
and has provided: product photo, competitor PDF, folder path, language, brand name.

These inputs are stored as:
```
PRODUCT_PHOTO       = [path or attached file]
COMPETITOR_PDF      = [attached file]
PRODUCT_PHOTOS_FOLDER = [folder path]
OUTPUT_LANGUAGE     = [language code]
BRAND_NAME          = [brand name as it appears on packaging]
VISUAL_MODE         = [from original prompt — REPLICATE or CUSTOM]
```

## Automation rule

Internal verification gates are MANDATORY before every generate_image call.
Printing "preconditions ok" without explicit checkboxes = Rule 16 violation = STOP.

---

## HARD PRECONDITIONS — before Phase A starts, ALL must be true

Print this checklist and confirm each item:

```
[ ] PRODUCT_PHOTO is a path to a file with ONLY the clean isolated package
    Verified by: Read the file → confirm no hands, no props, no creative overlay
    If not isolated → STOP. Tell user: "I need a clean packshot of just the packaging
    on white/transparent background. Save it to disk and give me the path."

[ ] Every file in PRODUCT_PHOTOS_FOLDER has been Read and listed in a table (see A2)

[ ] Every file is tagged: clean_packshot / lifestyle_with_pouch / design_composition / section_artwork

[ ] VISUAL_MODE is resolved → PACKAGING_BRAND decision made (see VISUAL_MODE rule below)
```

If any box is unchecked → STOP and surface what's missing. Never proceed with broken preconditions.

---

## VISUAL_MODE → PACKAGING_BRAND rule

```
If VISUAL_MODE = REPLICATE:
  PACKAGING_BRAND = competitor's packaging (keep original label, logo, text exactly as in reference)
  Higgsfield prompt includes: "product packaging preserves all original label artwork,
  logo, colors, badges, and text exactly as shown in the reference image"

If VISUAL_MODE = CUSTOM:
  PACKAGING_BRAND = user's brand
  Higgsfield prompt includes: "product packaging shows '[BRAND_NAME]' branding,
  all visible text in [OUTPUT_LANGUAGE]"
```

**BRAND_NAME replacement applies to LANDING COPY ONLY, never to packaging artwork when VISUAL_MODE = REPLICATE.**

---

## Phase A — Product gallery photos

### A1 — Upload PRODUCT_PHOTO to Higgsfield

```
1. Read PRODUCT_PHOTO file to confirm it's a clean packshot
2. Upload via Higgsfield media_upload tool
3. Confirm with media_confirm
4. Store returned media_id as PRODUCT_MEDIA_ID
```

### A2 — Inventory + scene analysis (HARD GATE)

Before ANY generate_image call, run `ls` on PRODUCT_PHOTOS_FOLDER and print this table:

```
| # | Filename | Tag | Composition | Background | Props | Text on image | Aspect | Goes to |
|---|----------|-----|-------------|------------|-------|---------------|--------|---------|
| 1 | hero.jpg | clean_packshot | centered pouch, straight-on | white | none | "BRAND NAME" label | 1:1 | gallery |
| 2 | hands.jpg | lifestyle_with_pouch | hand holding pouch at angle | cream | none | none | 1:1 | gallery |
| 3 | flatlay.jpg | lifestyle_with_pouch | pouch on marble with garlic | white marble | 3 garlic bulbs | none | 4:5 | gallery |
| 4 | benefits.jpg | section_artwork | 4-card benefits grid | dark red | icons | "Heart Health" etc | 16:9 | section-04 |
```

Tags:
- `clean_packshot` → isolated package, plain background, valid PRODUCT_PHOTO candidate
- `lifestyle_with_pouch` → package in a scene with props/hands/people → gallery image
- `design_composition` → creative artwork with text overlays → gallery image (keep as-is concept)
- `section_artwork` → image IS a landing section (benefits grid, comparison table, etc.) → Phase B

**Only after this table is printed are you allowed to call generate_image.**
**`section_artwork` files are NEVER uploaded as gallery images or used as Higgsfield reference directly.**

### PHASE A PRE-FLIGHT CHECKLIST (print verbatim before first generate_image)

```
[ ] A2 inventory table printed — one row per file in PRODUCT_PHOTOS_FOLDER
[ ] OUTPUT_LANGUAGE registered: ___
[ ] generations_planned = ___  ==  count(clean_packshot + lifestyle_with_pouch rows) = ___
[ ] For each row: A3 prompt block filled, text_overlays_output_lang field verified in OUTPUT_LANGUAGE
    (re-read each field — zero English words allowed if OUTPUT_LANGUAGE = es/fr/de/etc.)
[ ] PRODUCT_MEDIA_ID obtained and will be referenced in every prompt
[ ] VISUAL_MODE resolved → PACKAGING_BRAND rule decided
```

If ANY box is unchecked → STOP. Do NOT call generate_image.

### A3 — Strict prompt template (bilingual overlay, fill every field)

For each gallery image (clean_packshot or lifestyle_with_pouch), fill this template:

```
REFERENCE_FILE:               [exact filename — required]
COMPOSITION:                  [exact arrangement — pouch position, angle, what surrounds it]
BACKGROUND:                   [exact color + material: "warm cream linen" not "clean background"]
LIGHTING:                     [direction, hardness, color temp: "soft overhead warm light"]
PROPS_EXACT:                  [every prop: "3 raw garlic bulbs, fresh parsley sprig, 2 gold softgels"]
FOREGROUND_SUBJECTS:          [hands/people: "right hand of adult woman, no jewelry, beige skin tone"]
TEXT_OVERLAYS_SOURCE:         [verbatim text visible in reference image, in original language]
TEXT_OVERLAYS_OUTPUT_LANG:    [SAME text translated to OUTPUT_LANGUAGE — this goes in the prompt]
BADGES_CALLOUTS:              [every badge with label and position: "circular seal top-right: 'GARANTÍA 30 DÍAS'"]
POUCH_TREATMENT:              [from VISUAL_MODE rule]
ASPECT_RATIO:                 [match reference: 1:1 / 4:5 / 3:4 / 16:9]
```

**Two overlay fields are required.** Text in the final Higgsfield prompt uses TEXT_OVERLAYS_OUTPUT_LANG only.
If OUTPUT_LANGUAGE = es and the source has "MONEY BACK 30 DAYS" → TEXT_OVERLAYS_OUTPUT_LANG = "GARANTÍA DEVOLUCIÓN 30 DÍAS".

**Forbidden phrases** (replace with specifics):
- "minimal composition" → list exact items
- "clean background" → specify color and material
- "supplement bottle" → describe actual package shape

### A4 — Generate in Higgsfield

```
Model: nano_banana_2
Medias: [{ value: PRODUCT_MEDIA_ID, role: "image" }]
Prompt: [assembled from A3 template]
Aspect ratio: [from A3 fields]
```

### A4.5 — Mandatory visual verification (before ANY Shopify upload)

After every generate_image completes:

Print this block verbatim for each generated image:

```
VERIFICATION [reference filename]:
  Composition matches reference?    [YES/NO — 1 line description]
  Pouch artwork preserved?          [YES/NO]
  Overlays in OUTPUT_LANGUAGE only? [YES/NO — list any non-OUTPUT_LANGUAGE words found]
  → PASS (upload) | FAIL (regenerate) | DIVERGENCE (stop + report)
```

If this block is not printed → upload is not allowed.

3. If ANY answer is NO:
   - Do NOT upload to Shopify
   - Fix the prompt (most common fix: stronger pouch_treatment + verify PRODUCT_MEDIA_ID is correct)
   - Retry once
   - After 2 failed retries → STOP. Print DIVERGENCE REPORT:
     ```
     DIVERGENCE: [reference filename]
     Expected: [what the reference shows]
     Generated: [what came out — URL]
     Likely cause: [wrong VISUAL_MODE / generic prompt / wrong media reference]
     ```
     Wait for user input. Do not silently continue.

### A5 — Upload to Shopify product gallery

```bash
curl -X POST "https://STORE_URL/admin/api/2024-01/products/PRODUCT_ID/images.json" \
  -H "X-Shopify-Access-Token: ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"image": {"src": "HIGGSFIELD_CDN_URL", "alt": "PRODUCT_TITLE"}}'
```

Set hero shot as position 1. Upload all results automatically.

### A6 — Summary

```
✅ Product photos uploaded:
  1. [filename] → [type] → [Shopify image URL]
  2. ...
```

---

## Phase B — Landing section images

### HARD PRECONDITIONS for Phase B

Print and confirm before B2:

```
[ ] B1 inventory table printed (one row per image slot across all sections)
[ ] Image slot manifest printed (from section schema audit)
[ ] VISUAL STYLE GUIDE extracted from competitor PDF (see below)
[ ] Aspect ratio read from CSS for each slot and matched to Higgsfield ratio
```

### Visual Style Guide (extract from competitor PDF in Step 1)

Print this block before Step 6 and prepend it to EVERY Phase B prompt:

```
VISUAL_STYLE_GUIDE:
  people:
    age_range:     [e.g. 55-75]
    expression:    [warm / neutral / trusting]
    clothing:      [muted earth tones / cream / grey]
  settings:
    locations:     [home kitchen / living room / outdoor / lab]
    lighting:      [warm window light / soft overhead / golden hour]
    surfaces:      [warm wood / cream marble / linen]
  photo_style:     [editorial clean / commercial polished / film-like]
  color_palette:   [dominant colors across competitor images]
```

### B1 — Image inventory (HARD GATE)

Print this table before ANY B2 generation:

```
| Section file | Slot type | Schema key | PDF page ref | Image type | Subject | Aspect | Count needed |
|---|---|---|---|---|---|---|---|
| section-01-benefits.liquid | block | image_url | p.1, 4-card row | lifestyle | pouch + ingredient | 1:1 | 4 |
| section-03-split.liquid | section | image_url | p.2, left panel | product | pouch on surface | 1:1 | 1 |
| section-07-testimonials.liquid | block | ugc_url | p.3, carousel | UGC portrait | customer holding pouch | 1:1 | 8 |
...
```

Total generations needed for Phase B: [sum of Count needed column]

**Video thumbnails:** if ▶ visible in PDF → mark as `video_thumbnail`. Generate the still scene as a photo. Add ▶ overlay via CSS in the section (NOT in the Higgsfield prompt — icons in generated images render badly).

### B2 — Prompt template (style-guided)

Every Phase B prompt starts with the VISUAL_STYLE_GUIDE prefix, then adds:

```
[PHOTO_STYLE], [lighting], color palette dominated by [clothing + surface colors],
[per-image specifics from B1: subject, composition, props, aspect ratio]
```

### B2.5 — Multi-portrait sections (UGC carousels, review avatars)

For N distinct people needed, write N character cards before generating:

```
character_1: age 68, white-haired Latino man, navy henley, kitchen, holding pouch in left hand
character_2: age 55, blonde woman, beige cardigan, living room with plants, pouch in right hand
character_3: age 72, grey-haired African-American woman, blue blouse, outdoor patio, pouch on lap
```

Generate one prompt per character. No batching. After generation, verify they look like distinct people.
If two look like the same person → regenerate the duplicate with more distinct character description.

### B3 — Generate in Higgsfield

```
Model: nano_banana_2
Medias: [] (no product reference unless section shows product)
Prompt: [VISUAL_STYLE_GUIDE prefix] + [per-image specifics]
Aspect ratio: must match CSS slot aspect-ratio exactly
```

CSS → Higgsfield ratio mapping:
```
CSS aspect-ratio: 1/1  → "1:1"
CSS aspect-ratio: 4/5  → "4:5"
CSS aspect-ratio: 16/9 → "16:9"
CSS aspect-ratio: 3/4  → "3:4"
```

Read the section .liquid to confirm ratio before generating. Do not assume.

### B4.0 — Persist to Shopify Files (mandatory before injection)

Higgsfield URLs are temporary. For every Phase B image:

1. Upload to Shopify Files via stagedUploadsCreate + fileCreate (GraphQL)
2. Wait for Shopify CDN URL (cdn.shopify.com/...)
3. Use ONLY the Shopify CDN URL in template JSON

If stagedUploadsCreate fails → STOP and report. Do not inject a Higgsfield URL directly.

### B4 — Inject URLs (block-aware)

Two injection paths:

```
Case A — section-level slot:
  Path: templates/product.X.json → sections.<id>.settings.image_url
  Value: Shopify CDN URL

Case B — block-level slot (card grids, carousels, avatars):
  Path: templates/product.X.json → sections.<id>.blocks.<block_id>.settings.image_url
  Value: Shopify CDN URL
```

After injecting, read the JSON and grep for `image_url` — every slot from B1 must have a real https URL. None blank. Never share one URL across multiple distinct block slots.

### B5 — End-of-Phase-B verification (HARD GATE)

After all injections, fetch the storefront preview and verify:

For each section in B1 inventory:
- Count `<img>` tags in that section's HTML
- Must match the "Count needed" from B1 table
- Every `<img src>` must start with `https://cdn.shopify.com` (not Higgsfield CDN)

Any mismatch → STOP. Report missing slots. Fix before Step 7 QA.

### B6 — Summary

```
✅ Landing section images complete:
  section-01: 4/4 images injected ✓
  section-03: 1/1 image injected ✓
  section-07: 8/8 UGC + 3/3 avatars injected ✓
  ...
Total: [N]/[N] slots filled
```

---

## Execution order (automatic, no user stops)

```
Landing sections complete
    ↓
PRECONDITIONS CHECK (print checklist)
    ↓
Phase A — Product photos
  A1 → A2 (table) → A3 (prompt) → A4 (generate) → A4.5 (verify) → A5 (upload)
    ↓
Phase B — Landing section images
  PRECONDITIONS → B1 (inventory) → B2/B2.5 (prompts) → B3 (generate)
  → B4.0 (persist) → B4 (inject) → B5 (verify)
    ↓
Final summary
```
